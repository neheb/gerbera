project('lastfmlib', 'c', 'cpp',
  version : '0.4.0',
  default_options : ['warning_level=3', 'cpp_std=c++98'],
)

curl_dep = dependency('libcurl')
thread_dep = dependency('threads')

lastfmlib = library('lastfmlib',
  'lastfmlib/nowplayinginfo.cpp',
  'lastfmlib/urlclient.cpp',
  'lastfmlib/submissioninfocollection.cpp',
  'lastfmlib/lastfmscrobbler.cpp',
  'lastfmlib/submissioninfo.cpp',
  'lastfmlib/lastfmclient.cpp',
  'lastfmlib/lastfmscrobblerc.cpp',
  'lastfmlib/md5/md5.c',
  'lastfmlib/utils/scopedlock.cpp',
  'lastfmlib/utils/log.cpp',
  'lastfmlib/utils/thread.cpp',
  'lastfmlib/utils/condition.cpp',
  'lastfmlib/utils/mutex.cpp',
  'lastfmlib/utils/stringoperations.cpp',
  'lastfmlib/utils/numericoperations.cpp',
   dependencies : [ curl_dep, thread_dep ],
   install : true,
)

pkg = import('pkgconfig')
pkg.generate(lastfmlib)

install_headers(
  'lastfmlib/lastfmscrobbler.h',
  'lastfmlib/lastfmtypes.h',
  'lastfmlib/lastfmclient.h',
  'lastfmlib/lastfmscrobblerc.h',
  'lastfmlib/nowplayinginfo.h',
  'lastfmlib/submissioninfocollection.h',
  'lastfmlib/urlclient.h',
  'lastfmlib/submissioninfo.h',
  'lastfmlib/lastfmexceptions.h',
  subdir : 'lastfmlib',
)

install_headers(
  'lastfmlib/utils/log.h',
  'lastfmlib/utils/types.h',
  'lastfmlib/utils/condition.h',
  'lastfmlib/utils/scopedlock.h',
  'lastfmlib/utils/mutex.h',
  'lastfmlib/utils/thread.h',
  subdir : 'lastfmlib/utils'
)

lastfm_inc = include_directories('.')

lastfm_dep = declare_dependency(
  include_directories : lastfm_inc,
  link_with : lastfmlib,
)
