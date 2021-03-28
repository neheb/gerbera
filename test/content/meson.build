testcontent = executable(
  'testcontent',
  'main.cc',
  'test_autoscan_list.cc',
  'test_resolution.cc',
  cpp_args : libgerbera_args,
  dependencies : libgerbera_deps,
)

test('testcontent', testcontent)
