project('gerbera', 'c', 'cpp',
  version : '1.8.2',
  default_options : [ 'c_std=gnu11', 'cpp_std=c++17', 'warning_level=1' ],
)

cpp = meson.get_compiler('cpp')
if cpp.get_id() == 'gcc' and cpp.version().version_compare('<8')
  warning('Your GCC version is too old.  You need at least version 8')
elif cpp.get_id() == 'clang' and cpp.version().version_compare('<7')
  warning('Your clang version is too old.  You need at least version 7.')
endif

add_global_arguments('-DGERBERA_VERSION="@0@"'.format(meson.project_version()), language : 'cpp')
add_global_arguments('-DPACKAGE_DATADIR="@0@/@1@/@2@"'.format(get_option('prefix'), get_option('datadir'), meson.project_name()), language : 'cpp')

fs_dep = cpp.find_library('stdc++fs')
if not fs_dep.found()
  fs_dep = declare_dependency()
endif

fmt_dep = dependency('fmt', version : '>=5.3')
pugixml_dep = dependency('pugixml')
spdlog_dep = dependency('spdlog')
sqlite3_dep = dependency('sqlite3', version : '>=3.7.0')

iconv_dep = cpp.find_library('iconv', required : false)
if not iconv_dep.found()
  if cpp.has_function('iconv')
    iconv_dep = declare_dependency()
  else
    error('iconv implementation is needed')
  endif
endif

libgerbera_args = []

uuid_dep = dependency('uuid', required : false)
if not uuid_dep.found()
  if host_machine.system() == 'linux'
    error('libuuid is required')
  else
    libgerbera_args += '-DBSD_NATIVE_UUID'
  endif
endif

if host_machine.system() == 'solaris'
  libgerbera_args += '-DSOLARIS'
elif host_machine.system() != 'linux'
  libgerbera_args += '-DBSD'
endif

if get_option('buildtype') != 'debug'
    libgerbera_args += '-DSPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_INFO'
else
    libgerbera_args += '-DTOMBDEBUG'
    libgerbera_args += '-DSPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_DEBUG'
endif

if get_option('avcodec')
  avcodec_dep = dependency('libavcodec')
  avfilter_dep = dependency('libavfilter')
  avformat_dep = dependency('libavformat')
  avutil_dep = dependency('libavutil')
  libgerbera_args += '-DHAVE_FFMPEG'
  libgerbera_args += '-DHAVE_AVSTREAM_CODECPAR' #FIXME
else
  avcodec_dep = declare_dependency()
  avfilter_dep = declare_dependency()
  avformat_dep = declare_dependency()
  avutil_dep = declare_dependency()
endif

if get_option('curl')
  curl_dep = dependency('libcurl')
  libgerbera_args += '-DHAVE_CURL'
  libgerbera_args += '-DONLINE_SERVICES'
  libgerbera_args += '-DATRAILERS'
  libgerbera_args += '-DSOPCAST'
else
  curl_dep = declare_dependency()
endif

if get_option('exif')
  exif_dep = dependency('libexif')
  libgerbera_args += '-DHAVE_LIBEXIF'
else
  exif_dep = declare_dependency()
endif

if get_option('exiv2')
  exiv2_dep = dependency('exiv2')
  libgerbera_args += '-DHAVE_EXIV2'
else
  exiv2_dep = declare_dependency()
endif

if get_option('ffmpegthumbnailer')
  ffmpegthumbnailer_dep = dependency('libffmpegthumbnailer')
  libgerbera_args += '-DHAVE_FFMPEGTHUMBNAILER'
  if ffmpegthumbnailer_dep.version() > '2.2.0'
    libgerbera_args += '-DHAVE_FFMPEGTHUMBNAILER_SIZE_API'
  endif
else
  ffmpegthumbnailer_dep = declare_dependency()
endif

if get_option('inotify')
  if host_machine.system() == 'linux'
    libgerbera_args += '-DHAVE_INOTIFY'
  else
    error('inotify is Linux only. Please disable.')
  endif
endif

if get_option('js')
  duktape_dep = dependency('duktape')
  libgerbera_args += '-DHAVE_JS'
else
  duktape_dep = declare_dependency()
endif

if get_option('lastfm')
  lastfm_dep = dependency('lastfm')
  libgerbera_args += '-DHAVE_LASTFMLIB'
else
  lastfm_dep = declare_dependency()
endif

if get_option('magic')
  magic_dep = cpp.find_library('magic')
  libgerbera_args += '-DHAVE_MAGIC'
else
  magic_dep = declare_dependency()
endif

if get_option('matroska')
  ebml_dep = dependency('libebml')
  matroska_dep = dependency('libmatroska')
  libgerbera_args += '-DHAVE_MATROSKA'
else
  ebml_dep = declare_dependency()
  matroska_dep = declare_dependency()
endif

if get_option('mysql')
  mysql_dep = dependency('libmariadb')
  libgerbera_args += '-DHAVE_MYSQL'
else
  mysql_dep = declare_dependency()
endif

if get_option('npupnp')
  upnp_dep = dependency('libnpupnp')
  libgerbera_args += '-DUSING_NPUPNP'
else
  upnp_dep = dependency('libupnp')
endif

if get_option('taglib')
  taglib_dep = dependency('taglib')
  libgerbera_args += '-DHAVE_TAGLIB'
else
  taglib_dep = declare_dependency()
endif

libgerbera_deps = [
  avcodec_dep,
  avfilter_dep,
  avformat_dep,
  avutil_dep,
  curl_dep,
  duktape_dep,
  ebml_dep,
  exif_dep,
  exiv2_dep,
  fs_dep,
  fmt_dep,
  ffmpegthumbnailer_dep,
  iconv_dep,
  lastfm_dep,
  magic_dep,
  matroska_dep,
  mysql_dep,
  pugixml_dep,
  spdlog_dep,
  sqlite3_dep,
  taglib_dep,
  upnp_dep,
  uuid_dep,
]

# mangix hacks
add_global_arguments('-DCOMPILE_INFO="mangix"', language : 'cpp')
add_global_arguments('-DGIT_COMMIT_HASH="mangix"', language : 'cpp')
add_global_arguments('-DGIT_BRANCH="mangix"', language : 'cpp')

libgerbera_incdir = include_directories(['src'])

libgerbera = static_library('gerbera',
  'src/action_request.cc',
  'src/action_request.h',
  'src/cds_objects.cc',
  'src/cds_objects.h',
  'src/cds_resource.cc',
  'src/cds_resource.h',
  'src/common.h',
  'src/config/client_config.h',
  'src/config/client_config.cc',
  'src/config/config_definition.h',
  'src/config/config_definition.cc',
  'src/config/config_generator.h',
  'src/config/config_generator.cc',
  'src/config/config_manager.cc',
  'src/config/config_manager.h',
  'src/config/config_options.cc',
  'src/config/config_options.h',
  'src/config/config_setup.cc',
  'src/config/config_setup.h',
  'src/config/directory_tweak.cc',
  'src/config/directory_tweak.h',
  'src/config/dynamic_content.cc',
  'src/config/dynamic_content.h',
  'src/content/autoscan.cc',
  'src/content/autoscan.h',
  'src/content/autoscan_list.cc',
  'src/content/autoscan_list.h',
  'src/content/autoscan_inotify.cc',
  'src/content/autoscan_inotify.h',
  'src/content/content_manager.cc',
  'src/content/content_manager.h',
  'src/content/layout/builtin_layout.cc',
  'src/content/layout/builtin_layout.h',
  'src/content/layout/js_layout.cc',
  'src/content/layout/js_layout.h',
  'src/content/layout/layout.cc',
  'src/content/layout/layout.h',
  'src/content/onlineservice/atrailers_content_handler.cc',
  'src/content/onlineservice/atrailers_content_handler.h',
  'src/content/onlineservice/atrailers_service.cc',
  'src/content/onlineservice/atrailers_service.h',
  'src/content/onlineservice/curl_online_service.cc',
  'src/content/onlineservice/curl_online_service.h',
  'src/content/onlineservice/lastfm_scrobbler.cc',
  'src/content/onlineservice/lastfm_scrobbler.h',
  'src/content/onlineservice/online_service.cc',
  'src/content/onlineservice/online_service.h',
  'src/content/onlineservice/online_service_helper.cc',
  'src/content/onlineservice/online_service_helper.h',
  'src/content/onlineservice/sopcast_content_handler.cc',
  'src/content/onlineservice/sopcast_content_handler.h',
  'src/content/onlineservice/sopcast_service.cc',
  'src/content/onlineservice/sopcast_service.h',
  'src/content/onlineservice/task_processor.cc',
  'src/content/onlineservice/task_processor.h',
  'src/content/scripting/import_script.cc',
  'src/content/scripting/import_script.h',
  'src/content/scripting/js_functions.cc',
  'src/content/scripting/js_functions.h',
  'src/content/scripting/playlist_parser_script.cc',
  'src/content/scripting/playlist_parser_script.h',
  'src/content/scripting/scripting_runtime.cc',
  'src/content/scripting/scripting_runtime.h',
  'src/content/scripting/script.cc',
  'src/content/scripting/script.h',
  'src/content/scripting/script_names.h',
  'src/content/update_manager.cc',
  'src/content/update_manager.h',
  'src/context.cc',
  'src/context.h',
  'src/contrib/md5.c',
  'src/contrib/md5.h',
  'src/device_description_handler.cc',
  'src/device_description_handler.h',
  'src/exceptions.cc',
  'src/exceptions.h',
  'src/file_request_handler.cc',
  'src/file_request_handler.h',
  'src/iohandler/buffered_io_handler.cc',
  'src/iohandler/buffered_io_handler.h',
  'src/iohandler/curl_io_handler.cc',
  'src/iohandler/curl_io_handler.h',
  'src/iohandler/file_io_handler.cc',
  'src/iohandler/file_io_handler.h',
  'src/iohandler/io_handler_buffer_helper.cc',
  'src/iohandler/io_handler_buffer_helper.h',
  'src/iohandler/io_handler.cc',
  'src/iohandler/io_handler_chainer.cc',
  'src/iohandler/io_handler_chainer.h',
  'src/iohandler/io_handler.h',
  'src/iohandler/mem_io_handler.cc',
  'src/iohandler/mem_io_handler.h',
  'src/iohandler/process_io_handler.cc',
  'src/iohandler/process_io_handler.h',
  'src/metadata/exiv2_handler.cc',
  'src/metadata/exiv2_handler.h',
  'src/metadata/ffmpeg_handler.cc',
  'src/metadata/ffmpeg_handler.h',
  'src/metadata/metadata_handler.cc',
  'src/metadata/metadata_handler.h',
  'src/metadata/libexif_handler.cc',
  'src/metadata/libexif_handler.h',
  'src/metadata/taglib_handler.cc',
  'src/metadata/taglib_handler.h',
  'src/metadata/metacontent_handler.cc',
  'src/metadata/metacontent_handler.h',
  'src/metadata/matroska_handler.cc',
  'src/metadata/matroska_handler.h',
  'src/request_handler.cc',
  'src/request_handler.h',
  'src/server.cc',
  'src/serve_request_handler.cc',
  'src/serve_request_handler.h',
  'src/server.h',
  'src/database/mysql/mysql_database.cc',
  'src/database/mysql/mysql_database.h',
  'src/database/sqlite3/sqlite_database.cc',
  'src/database/sqlite3/sqlite_database.h',
  'src/database/sql_database.cc',
  'src/database/sql_database.h',
  'src/database/database.cc',
  'src/database/database.h',
  'src/database/search_handler.cc',
  'src/database/search_handler.h',
  'src/subscription_request.cc',
  'src/subscription_request.h',
  'src/transcoding/transcode_dispatcher.cc',
  'src/transcoding/transcode_dispatcher.h',
  'src/transcoding/transcode_ext_handler.cc',
  'src/transcoding/transcode_ext_handler.h',
  'src/transcoding/transcode_handler.cc',
  'src/transcoding/transcode_handler.h',
  'src/transcoding/transcoding.cc',
  'src/transcoding/transcoding.h',
  'src/transcoding/transcoding_process_executor.cc',
  'src/transcoding/transcoding_process_executor.h',
  'src/upnp_cds.cc',
  'src/upnp_cds.h',
  'src/upnp_cm.cc',
  'src/upnp_cm.h',
  'src/upnp_mrreg.cc',
  'src/upnp_mrreg.h',
  'src/upnp_xml.cc',
  'src/upnp_xml.h',
  'src/url_request_handler.cc',
  'src/url_request_handler.h',
  'src/util/executor.h',
  'src/util/generic_task.cc',
  'src/util/generic_task.h',
  'src/util/jpeg_resolution.cc',
  'src/util/logger.h',
  'src/util/mime.cc',
  'src/util/mime.h',
  'src/util/mt_inotify.cc',
  'src/util/mt_inotify.h',
  'src/util/process.cc',
  'src/util/process_executor.cc',
  'src/util/process_executor.h',
  'src/util/process.h',
  'src/util/string_converter.cc',
  'src/util/string_converter.h',
  'src/util/thread_executor.cc',
  'src/util/thread_executor.h',
  'src/util/thread_runner.h',
  'src/util/timer.cc',
  'src/util/timer.h',
  'src/util/tools.cc',
  'src/util/tools.h',
  'src/util/upnp_clients.h',
  'src/util/upnp_clients.cc',
  'src/util/upnp_headers.h',
  'src/util/upnp_headers.cc',
  'src/util/upnp_quirks.h',
  'src/util/upnp_quirks.cc',
  'src/util/url.cc',
  'src/util/url.h',
  'src/util/xml_to_json.cc',
  'src/util/xml_to_json.h',
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
  'src/web/pages.h',
  'src/web/remove.cc',
  'src/web/web_request_handler.cc',
  'src/web/web_request_handler.h',
  'src/web/session_manager.cc',
  'src/web/session_manager.h',
  'src/web/tasks.cc',
  'src/web/web_autoscan.cc',
  'src/web/web_update.cc',
  include_directories : libgerbera_incdir,
  dependencies : libgerbera_deps,
  cpp_args : libgerbera_args,
)

gerbera = executable('gerbera',
  'src/main.cc',
  include_directories : libgerbera_incdir,
  link_with : libgerbera,
  dependencies : libgerbera_deps,
  install : true,
)


