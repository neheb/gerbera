testcontent = executable(
  'testcontent',
  'main.cc',
  'test_autoscan_timed.cc',
  cpp_args : libgerbera_args,
  dependencies : libgerbera_deps,
  include_directories : libgerbera_incdir,
  link_with : libgerbera,
)

test('testcontent', testcontent)
