!SLIDE

# Tracing `pip install` #

!SLIDE small

# `pip/commands/install.py` #

    @@@ python
    class InstallCommand(Command):
        name = 'install'
        usage = '%prog [OPTIONS] PACKAGE_NAMES...'
        summary = 'Install packages'
        bundle = False

        def __init__(self):
            super(InstallCommand, self).__init__()
            self.parser.add_option(
                '-e', '--editable',
                dest='editables',
                action='append',
                default=[],
                metavar='VCS+REPOS_URL[@REV]#egg=PACKAGE',
        # ...

!SLIDE smaller

    @@@ python
    class InstallCommand(Command):
        # ...
        def run(self, options, args):
            # ... skip a bunch of options handling ...
            finder = self._build_package_finder(options, index_urls)

            requirement_set = RequirementSet(
                build_dir=options.build_dir,
                src_dir=options.src_dir,
                download_dir=options.download_dir,
                download_cache=options.download_cache,
                upgrade=options.upgrade,
                ignore_installed=options.ignore_installed,
                ignore_dependencies=options.ignore_dependencies)
            for name in args:
                requirement_set.add_requirement(
                    InstallRequirement.from_line(name, None))
            for name in options.editables:
                requirement_set.add_requirement(
                    InstallRequirement.from_editable(
                        name, default_vcs=options.default_vcs))
            for filename in options.requirements:
                for req in parse_requirements(
                    filename, finder=finder, options=options):
                    requirement_set.add_requirement(req)

!SLIDE smaller

    @@@ python
    class InstallCommand(Command):
        # ...
        def run(self, options, args):
            # ...
            if not options.no_download:
                requirement_set.prepare_files(
                    finder,
                    force_root_egg_info=self.bundle,
                    bundle=self.bundle)
            else:
                requirement_set.locate_files()

            if not options.no_install and not self.bundle:
                requirement_set.install(
                    install_options, global_options)
            elif self.bundle:
                requirement_set.create_bundle(self.bundle_filename)
            # Clean up
            if not options.no_install:
                requirement_set.cleanup_files(bundle=self.bundle)
            return requirement_set

!SLIDE smbullets incremental

## `RequirementSet.prepare_files()` ##

* Finds and downloads source distribution for all requirements.
* Unpacks to temporary location, runs `setup.py egg-info` (using the
  setuptools-import hack) to generate metadata, and resolves dependencies.
* This method, as Ian so delicately puts it, "grew organically."
* When it's done, we expect `self.requirements` to be a dictionary mapping
  project names to `InstallRequirement` objects that are fully prepared for
  installation.

!SLIDE smaller

    @@@ python
    class RequirementSet(object):
        # ...
        def install(self, install_options, global_options=()):
            """Install everything in this set
            (after having downloaded and unpacked the packages)"""
            to_install = [r for r in self.requirements.values()
                          if self.upgrade or not r.satisfied_by]

            for requirement in to_install:
                if requirement.conflicts_with:
                    requirement.uninstall(auto_confirm=True)
                try:
                    requirement.install(install_options, global_options)
                except:
                    # if install did not succeed, rollback uninstall
                    if (requirement.conflicts_with and
                        not requirement.install_succeeded):
                        requirement.rollback_uninstall()
                    raise
                else:
                    if (requirement.conflicts_with and
                        requirement.install_succeeded):
                        requirement.commit_uninstall()
                requirement.remove_temporary_source()
            self.successfully_installed = to_install

!SLIDE small

    @@@ python
    class InstallRequirement(object):
        # ...
        def install(self, install_options, global_options=()):
            # ...
            install_args = [
                sys.executable, '-c',
                "import setuptools;__file__=%r;"\
                "execfile(__file__)" % self.setup_py] +\
                list(global_options) + [
                'install',
                '--single-version-externally-managed',
                '--record', record_filename]

            # ...

            call_subprocess(install_args + install_options,
                cwd=self.source_dir,
                filter_stdout=self._filter_install,
                show_stdout=False)

