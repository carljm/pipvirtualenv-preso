!SLIDE

# Using `pip` as an API #

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

        def run(self, options, args):
            # ...

!SLIDE small

    @@@ python
    from pip.req import InstallRequirement, RequirementSet
    from pip.index import PackageFinder
    from pip.locations import build_prefix, src_prefix
    reqset = RequirementSet(build_dir=build_prefix,
                            src_dir=src_prefix,
                            download_dir=None)
    req = InstallRequirement.from_line("yolk==0.4.1",
                                       comes_from=None)
    reqset.add_requirement(req)
    finder = PackageFinder(
        find_links=[],
        index_urls=["http://pypi.python.org/simple"])
    reqset.prepare_files(finder)
    reqset.install(install_options=[], global_options=[])
    reqset.cleanup_files()
