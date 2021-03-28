project(
  'gerbera',
  'c',
  'cpp',
  version : '1.11.0',
  default_options : [ 'cpp_std=c++17', 'warning_level=1', 'default_library=static' ],
  meson_version : '>=0.47',
)

cpp = meson.get_compiler('cpp')
if cpp.get_id() == 'gcc' and cpp.version().version_compare('<8.4')
  error('Your GCC version is too old. You need at least version 8.4.')
elif cpp.get_id() == 'clang' and cpp.version().version_compare('<7')
  error('Your clang version is too old. You need at least version 7.')
endif

add_project_arguments(
  '-DGERBERA_VERSION="@0@"'.format(meson.project_version()),
  '-DPACKAGE_DATADIR="@0@"'.format(
    join_paths(get_option('prefix'), get_option('datadir'), meson.project_name()),
  ),
  language : 'cpp',
)

# This makes assumptions that the libcpp is the GNU one on Linux systems
if cpp.version().version_compare('>=9')
  fs_dep = dependency('', required : false)
elif host_machine.system() == 'linux'
  fs_dep = cpp.find_library('stdc++fs')
elif cpp.get_id() == 'clang'
  fs_dep = cpp.find_library('c++fs')
endif

cxxopts_dep = dependency('cxxopts')
fmt_dep = dependency('fmt', version : '>=8')
pugixml_dep = dependency('pugixml')
spdlog_dep = dependency('spdlog', version : '>=1.8.2')
sqlite3_dep = dependency('sqlite3', version : '>=3.7.11')

iconv_dep = cpp.find_library('iconv', required : false)
if not iconv_dep.found()
  if cpp.has_function('iconv')
    iconv_dep = dependency('', required : false)
  else
    error('iconv implementation is needed')
  endif
endif

if cpp.links(
  '''
  #include <stdint.h>
  int main(void) {
    uint32_t x32 = 0;
    uint64_t x64 = 0;
    __atomic_load_n(&x32, __ATOMIC_SEQ_CST);
    __atomic_load_n(&x64, __ATOMIC_SEQ_CST);
    return 0;
    }''',
  name : 'built-in atomics',
)
  libatomic_dep = dependency('', required : false)
else
  libatomic_dep = cpp.find_library('atomic')
endif

libgerbera_args = []

if (cpp.has_function('nl_langinfo'))
  libgerbera_args += '-DHAVE_NL_LANGINFO'
endif

if (cpp.has_function('setlocale'))
  libgerbera_args += '-DHAVE_SETLOCALE'
endif

uuid_dep = dependency('uuid', required : host_machine.system() == 'linux')
if not uuid_dep.found()
  libgerbera_args += '-DBSD_NATIVE_UUID'
endif

nsl_dep = dependency('nsl', required : host_machine.system() == 'sunos')
socket_dep = dependency('socket', required : host_machine.system() == 'sunos')

if host_machine.system() == 'sunos'
  libgerbera_args += '-DSOLARIS'
elif host_machine.system() != 'linux'
  libgerbera_args += '-DBSD'
endif

if get_option('debug')
  libgerbera_args += '-DTOMBDEBUG'
  libgerbera_args += '-DSPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_DEBUG'
else
  libgerbera_args += '-DSPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_INFO'
endif

avcodec_dep = dependency('libavcodec', required : get_option('avcodec'))
avfilter_dep = dependency('libavfilter', required : get_option('avcodec'))
avformat_dep = dependency('libavformat', required : get_option('avcodec'))
avutil_dep = dependency('libavutil', required : get_option('avcodec'))
if avcodec_dep.found() and avfilter_dep.found() and avformat_dep.found() and avutil_dep.found()
  libgerbera_args += '-DHAVE_FFMPEG'
  if cpp.has_member(
    'struct AVStream',
    'codecpar',
    prefix : '#include <libavformat/avformat.h>',
    dependencies : avformat_dep,
  )
    libgerbera_args += '-DHAVE_AVSTREAM_CODECPAR'
  endif
endif

curl_dep = dependency('libcurl', required : get_option('curl'))
if curl_dep.found()
  libgerbera_args += '-DHAVE_CURL'
  libgerbera_args += '-DONLINE_SERVICES'
  libgerbera_args += '-DATRAILERS'
endif

exif_dep = dependency('libexif', required : get_option('exif'))
if exif_dep.found()
  libgerbera_args += '-DHAVE_LIBEXIF'
endif

exiv2_dep = dependency('exiv2', required : get_option('exiv2'))
if exiv2_dep.found()
  libgerbera_args += '-DHAVE_EXIV2'
  if exiv2_dep.version().version_compare('<0.28')
    libgerbera_args += '-D_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR'
  endif
endif

ffmpegthumbnailer_dep = dependency('libffmpegthumbnailer', required : get_option('ffmpegthumbnailer'))
if ffmpegthumbnailer_dep.found()
  libgerbera_args += '-DHAVE_FFMPEGTHUMBNAILER'
endif

if host_machine.system() == 'linux' or host_machine.system() == 'sunos'
  inotify_dep = dependency('', required : false)
  inotify_found = not get_option('inotify').disabled()
else
  inotify_dep = dependency('inotify', required : get_option('inotify'))
  inotify_found = inotify_dep.found()
endif

if inotify_found
  libgerbera_args += '-DHAVE_INOTIFY'
endif

duktape_dep = dependency('duktape', required : get_option('js'))
if duktape_dep.found()
  libgerbera_args += '-DHAVE_JS'
endif

lastfm_dep = dependency('liblastfm', required : get_option('lastfm'))
if lastfm_dep.found()
  libgerbera_args += '-DHAVE_LASTFMLIB'
endif

magic_dep = cpp.find_library('magic', required : get_option('magic'))
if magic_dep.found()
  libgerbera_args += '-DHAVE_MAGIC'
endif

ebml_dep = dependency('libebml', required : get_option('matroska'))
matroska_dep = dependency('libmatroska', required : get_option('matroska'))
if ebml_dep.found() and matroska_dep.found()
  libgerbera_args += '-DHAVE_MATROSKA'
endif

mysql_dep = dependency('libmariadb', required : get_option('mysql'))
if mysql_dep.found()
  libgerbera_args += '-DHAVE_MYSQL'
endif

upnp_dep = dependency('libnpupnp', version : '>= 4.2.1', required : get_option('npupnp'))
if upnp_dep.found()
  libgerbera_args += '-DUSING_NPUPNP'
else
  upnp_dep = dependency('libupnp', version : '>= 1.14.6')
endif

taglib_dep = dependency('taglib', version : '>= 1.12', required : get_option('taglib'))
if taglib_dep.found()
  libgerbera_args += '-DHAVE_TAGLIB'
endif

gmock_dep = dependency('gmock', required : get_option('tests'))
gtest_dep = dependency('gtest', required : get_option('tests'))

libgerbera_deps = [
  avcodec_dep,
  avfilter_dep,
  avformat_dep,
  avutil_dep,
  cxxopts_dep,
  curl_dep,
  duktape_dep,
  ebml_dep,
  exif_dep,
  exiv2_dep,
  fs_dep,
  fmt_dep,
  ffmpegthumbnailer_dep,
  iconv_dep,
  inotify_dep,
  lastfm_dep,
  libatomic_dep,
  magic_dep,
  matroska_dep,
  mysql_dep,
  nsl_dep,
  pugixml_dep,
  socket_dep,
  spdlog_dep,
  sqlite3_dep,
  taglib_dep,
  upnp_dep,
  uuid_dep,
]

compile_info = [
  'WITH_MAGIC=' + magic_dep.found().to_string('YES', 'NO'),
  'WITH_MYSQL=' + mysql_dep.found().to_string('YES', 'NO'),
  'WITH_CURL=' + curl_dep.found().to_string('YES', 'NO'),
  'WITH_INOTIFY=' + inotify_found.to_string('YES', 'NO'),
  'WITH_JS=' + duktape_dep.found().to_string('YES', 'NO'),
  'WITH_TAGLIB=' + taglib_dep.found().to_string('YES', 'NO'),
  'WITH_AVCODEC=' + avcodec_dep.found().to_string('YES', 'NO'),
  'WITH_FFMPEGTHUMBNAILER=' + ffmpegthumbnailer_dep.found().to_string('YES', 'NO'),
  'WITH_EXIF=' + exif_dep.found().to_string('YES', 'NO'),
  'WITH_EXIV2=' + exiv2_dep.found().to_string('YES', 'NO'),
  'WITH_SYSTEMD=' + get_option('systemd').to_string('YES', 'NO'),
  'WITH_LASTFM=' + lastfm_dep.found().to_string('YES', 'NO'),
  'WITH_DEBUG=' + get_option('debug').to_string('YES', 'NO'),
  'WITH_TESTS=' + gtest_dep.found().to_string('YES', 'NO'),
]

add_project_arguments('-DCOMPILE_INFO="@0@"'.format('\\n'.join(compile_info)), language : 'cpp')

gchash = run_command('git', 'rev-parse', 'HEAD', check : false, capture : true)
if gchash.returncode() == 0
  add_project_arguments('-DGIT_COMMIT_HASH="@0@"'.format(gchash.stdout().strip()), language : 'cpp')
else
  add_project_arguments('-DGIT_COMMIT_HASH=""', language : 'cpp')
endif

gbranch = run_command('git', 'rev-parse', '--symbolic-full-name', 'HEAD', check : false, capture : true)
if gbranch.returncode() == 0
  add_project_arguments('-DGIT_BRANCH="@0@"'.format(gbranch.stdout().strip()), language : 'cpp')
else
  add_project_arguments('-DGIT_BRANCH=""', language : 'cpp')
endif

libgerbera_incdir = include_directories('src')

libgerbera = static_library(
  'gerbera',
  'src/action_request.cc',
  'src/cds_objects.cc',
  'src/cds_resource.cc',
  'src/config/client_config.cc',
  'src/config/config_definition.cc',
  'src/config/config_generator.cc',
  'src/config/config_manager.cc',
  'src/config/config_options.cc',
  'src/config/config_setup.cc',
  'src/config/directory_tweak.cc',
  'src/config/dynamic_content.cc',
  'src/content/autoscan.cc',
  'src/content/autoscan_list.cc',
  'src/content/autoscan_inotify.cc',
  'src/content/content_manager.cc',
  'src/content/layout/builtin_layout.cc',
  'src/content/layout/js_layout.cc',
  'src/content/onlineservice/atrailers_content_handler.cc',
  'src/content/onlineservice/atrailers_service.cc',
  'src/content/onlineservice/curl_online_service.cc',
  'src/content/onlineservice/lastfm_scrobbler.cc',
  'src/content/onlineservice/online_service.cc',
  'src/content/onlineservice/online_service_helper.cc',
  'src/content/onlineservice/task_processor.cc',
  'src/content/scripting/import_script.cc',
  'src/content/scripting/js_functions.cc',
  'src/content/scripting/playlist_parser_script.cc',
  'src/content/scripting/scripting_runtime.cc',
  'src/content/scripting/script.cc',
  'src/content/update_manager.cc',
  'src/context.cc',
  'src/contrib/md5.c',
  'src/device_description_handler.cc',
  'src/exceptions.cc',
  'src/file_request_handler.cc',
  'src/iohandler/buffered_io_handler.cc',
  'src/iohandler/curl_io_handler.cc',
  'src/iohandler/file_io_handler.cc',
  'src/iohandler/io_handler_buffer_helper.cc',
  'src/iohandler/io_handler.cc',
  'src/iohandler/io_handler_chainer.cc',
  'src/iohandler/mem_io_handler.cc',
  'src/iohandler/process_io_handler.cc',
  'src/metadata/exiv2_handler.cc',
  'src/metadata/ffmpeg_handler.cc',
  'src/metadata/libexif_handler.cc',
  'src/metadata/matroska_handler.cc',
  'src/metadata/metacontent_handler.cc',
  'src/metadata/metadata_handler.cc',
  'src/metadata/taglib_handler.cc',
  'src/request_handler.cc',
  'src/server.cc',
  'src/database/mysql/mysql_database.cc',
  'src/database/sqlite3/sqlite_database.cc',
  'src/database/sql_database.cc',
  'src/database/database.cc',
  'src/database/search_handler.cc',
  'src/subscription_request.cc',
  'src/transcoding/transcode_dispatcher.cc',
  'src/transcoding/transcode_ext_handler.cc',
  'src/transcoding/transcode_handler.cc',
  'src/transcoding/transcoding.cc',
  'src/transcoding/transcoding_process_executor.cc',
  'src/upnp_cds.cc',
  'src/upnp_cm.cc',
  'src/upnp_mrreg.cc',
  'src/upnp_xml.cc',
  'src/url_request_handler.cc',
  'src/util/generic_task.cc',
  'src/util/jpeg_resolution.cc',
  'src/util/mime.cc',
  'src/util/mt_inotify.cc',
  'src/util/process.cc',
  'src/util/process_executor.cc',
  'src/util/string_converter.cc',
  'src/util/thread_executor.cc',
  'src/util/timer.cc',
  'src/util/tools.cc',
  'src/util/upnp_clients.cc',
  'src/util/upnp_headers.cc',
  'src/util/upnp_quirks.cc',
  'src/util/url.cc',
  'src/util/xml_to_json.cc',
  'src/web/action.cc',
  'src/web/add.cc',
  'src/web/add_object.cc',
  'src/web/auth.cc',
  'src/web/clients.cc',
  'src/web/config_load.cc',
  'src/web/config_save.cc',
  'src/web/containers.cc',
  'src/web/directories.cc',
  'src/web/edit_load.cc',
  'src/web/edit_save.cc',
  'src/web/files.cc',
  'src/web/items.cc',
  'src/web/pages.cc',
  'src/web/remove.cc',
  'src/web/web_request_handler.cc',
  'src/web/session_manager.cc',
  'src/web/tasks.cc',
  'src/web/web_autoscan.cc',
  'src/web/web_update.cc',
  include_directories : libgerbera_incdir,
  dependencies : libgerbera_deps,
  cpp_args : libgerbera_args,
)

gerbera = executable(
  'gerbera',
  'src/main.cc',
  include_directories : libgerbera_incdir,
  link_with : libgerbera,
  dependencies : libgerbera_deps,
  install : true,
)

install_data(
  'src/database/mysql/mysql.sql',
  'src/database/mysql/mysql-upgrade.xml',
  'src/database/sqlite3/sqlite3.sql',
  'src/database/sqlite3/sqlite3-upgrade.xml',
)

install_subdir('scripts/js', install_dir : 'share/gerbera')
install_subdir('web', install_dir : 'share/gerbera')

install_man('doc/gerbera.1')

if gmock_dep.found() and gtest_dep.found()
  subdir('test')
endif
