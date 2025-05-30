git = find_program('git')
rpmbuild = find_program('rpmbuild')
tar = find_program('tar')
bash = find_program('bash')

commit = run_command(git, 'rev-parse', 'HEAD', capture: true, check: true).stdout().strip()

systemd_sysusers_dir = systemd.get_variable(pkgconfig: 'sysusersdir')

if get_option('vendor')
  run_command(
    [bash, files(meson.project_source_root() / 'scripts' / 'vendor.sh')],
    check: true,
  )
endif

build_root_basename = meson.project_build_root().split('/').get(-1)
tarball = custom_target(
  'tarball',
  command: [
    tar,
    '--create',
    '--gzip',
    '--absolute-names',
    '--file', '@OUTPUT@',
    '--transform', 's+@0@+@1@-@2@+'.format(
      meson.project_source_root(),
      meson.project_name(),
      meson.project_version(),
    ),
    '--exclude', build_root_basename,
    '--exclude', '.git',
    '--exclude', '.vscode',
    '--exclude', '.github',
    '--exclude', '.copr',
    meson.project_source_root(),
  ],
  output: '@0@-@1@.tar.gz'.format(meson.project_name(), meson.project_version()),
)

specfile = configure_file(
  input: 'rhc.spec.in',
  output: '@BASENAME@',
  configuration: {
    'COMMIT': commit,
    'VERSION': meson.project_version(),
  },
)

run_target(
  'srpm',
  command: [
    rpmbuild,
    '-bs',
    '--define', 'commit @0@'.format(commit),
    '--define', '_srcrpmdir @0@'.format(meson.current_build_dir()),
    '--define', '_sourcedir @0@'.format(meson.current_build_dir()),
    specfile,
  ],
  depends: [tarball],
)
