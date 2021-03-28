testutil = executable(
  'testutil',
  'main.cc',
  'test_jpeg_res.cc',
  'test_tools.cc',
  'test_upnp_clients.cc',
  'test_upnp_headers.cc',
  cpp_args : libgerbera_args,
  dependencies : libgerbera_deps,
)

test('testutil', testutil, workdir : meson.current_source_dir())
