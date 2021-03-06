project('RevBayes', ['cpp', 'c'],
        version: '1.0.10',
        default_options: [
          'buildtype=release',
          'cpp_std=c++14'
        ],
        meson_version: '>= 0.47',
        license: 'GPLv2')

cpp = meson.get_compiler('cpp')
cc = meson.get_compiler('c')


if get_option('buildtype') == 'release' or get_option('buildtype') == 'plain'
  add_project_arguments(['-O3','-msse','-msse2','-msse3'], language : 'cpp')
  add_project_arguments(['-O3'], language : 'c')
endif

if get_option('buildtype') == 'debug' or get_option('buildtype') == 'debugoptimized'
  # FIXME: -Woverloaded-virtual is useful, but has too many false-positives with GCC?
  add_project_arguments(['-Wall',
                         # These bugs are very hard to track down:
                         '-Wstrict-aliasing'],
                        language : 'cpp')
endif

# Disable some warnings
add_project_arguments([# These just add noise:
                       '-Wno-sign-compare','-Wno-unknown-pragmas',
                       # These could be bugs, but there are so many we can't see the really bad ones. So, disable them for now:
                       '-Wno-reorder',
                       # Not necessarily a bug:
                       '-Wno-unused-variable','-Wno-unused-but-set-variable'],
                      language : 'cpp')

# Handle file too large error when cross-compiling to windows
if host_machine.system() == 'windows'
  add_project_arguments(['-Wa,-mbig-obj'], language : 'cpp')
endif

boost_modules = [ 'regex',
                  'program_options',
                  'thread',
                  'system',
                  'filesystem',
                  'date_time',
                  'serialization']

want_static = get_option('static')

# This doesn't build an internal boost from the internal copy, yet.
boost = dependency('boost', modules : boost_modules, static: want_static)

rb_name = 'rb'
if get_option('mpi')
  add_project_arguments(['-DRB_MPI'], language: 'cpp')
  mpi = dependency('mpi', language: 'cpp', static: want_static)
else
  mpi = dependency('', required: false)
endif

if get_option('jupyter')
  rb_name += '-jupyter'
  add_project_arguments(['-DRB_XCODE'], language : 'cpp')
endif

####
subdir('src')

############# executables #################
core = static_library('rb-core',
                      core_sources,
                      include_directories: [src_inc],
                      dependencies: [mpi])

revlanguage = static_library('rb-revlanguage',
                             revlanguage_sources,
                             include_directories: [src_inc],
                             dependencies: [mpi])

libs = static_library('rb-lib',
                      libs_sources,
                      include_directories: [src_inc])

############# executables #################

maybe_link_static = []
if want_static
  maybe_link_static = ['-static']
endif

rb = executable(rb_name,
                ['src/revlanguage/main.cpp'],
                link_with: [core, revlanguage, libs],
                include_directories: [src_inc],
                dependencies: [boost, mpi],
                link_args: maybe_link_static,
                install: true)

if get_option('studio')
  gtk2 = dependency('gtk+-2.0', static: want_static)
  cmd = static_library('rb-cmd',
                       cmd_sources,
                       include_directories: [src_inc],
                       dependencies: [gtk2])

  #  Add cmd/main.cpp cmd/RbGTKGui.cpp
  #  includes: RbGTRGui.ui
  revstudio = executable('RevStudio',
                         ['src/cmd/main.cpp'],
                         link_with: [core, revlanguage, libs, cmd],
                         include_directories: [src_inc],
                         dependencies: [boost, mpi, gtk2],
                         install: true)

endif

subdir('tests')

