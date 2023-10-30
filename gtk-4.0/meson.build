# render PNG assets

gtk4_asset_names = run_command(
  'cat', 'assets' / 'assets.txt',
  check : true
).stdout().split()

subdir('assets')

# compile CSS

sass_path = meson.current_source_dir() / 'sass'
sass_depend_files = run_command(
  'find',
  sass_path,
  '-name', '_*.scss',
  check : true
).stdout().split()

# always compile the dark CSS
input_scss_dark = configure_file(
  input: meson.current_source_dir() / 'sass' / 'gtk.scss.in',
  output : 'gtk-dark.scss',
  configuration : {
    'variant' : 'dark',
    'transparency' : get_option('transparency').to_string(),
    'sass_path' : meson.current_source_dir() / 'sass',
  }
)

output_css_dark = 'gtk-main-dark.css'
gtk4_stylesheets = []

gtk4_stylesheets += custom_target(
  output_css_dark,
  input : input_scss_dark,
  output : output_css_dark,
  command : [sassc, sassc_options, '@INPUT@', '@OUTPUT@'],
  build_by_default : true,
  depend_files : sass_depend_files
)

foreach variant : get_option('variants')
  output_css = (variant == 'light' ? 'gtk-main.css' : 'gtk-main-' + variant + '.css')

  if variant != 'dark'
    input_scss = configure_file(
      input: meson.current_source_dir() / 'sass' / 'gtk.scss.in',
      output : 'gtk-' + variant + '.scss',
      configuration : {
        'variant' : variant,
        'transparency' : get_option('transparency').to_string(),
        'sass_path' : meson.current_source_dir() / 'sass',
      }
    )

    gtk4_stylesheets += custom_target(
      output_css,
      input : input_scss,
      output : output_css,
      command : [sassc, sassc_options, '@INPUT@', '@OUTPUT@'],
      build_by_default : true,
      depend_files : sass_depend_files
    )
  endif

  # generate the gresource XML

  gresource_xml_array = [
    '<?xml version="1.0" encoding="UTF-8"?>',
    '<gresources>',
    '<gresource prefix="/org/gnome/arc-theme">'
  ]

  foreach asset : gtk4_asset_names
    gresource_xml_array += [
      '<file>assets/' + asset + '.png</file>',
      '<file>assets/' + asset + '@2.png</file>'
    ]
  endforeach

  gresource_xml_array += ['<file>' + output_css + '</file>']

  if variant != 'dark'
    gresource_xml_array += ['<file>' + output_css_dark + '</file>']
  endif

  gresource_xml_array += [
    '</gresource>',
    '</gresources>'
  ]

  # compile the gresource

  gresource_xml = configure_file(
    capture : true,
    command : ['echo', gresource_xml_array],
    output : 'gtk-' + variant + '.gresource.xml'
  )

  #TODO use gnome.compile_resources()???
  gresource = custom_target(
    'gresource-' + variant,
    input : gresource_xml,
    output : 'gtk-' + variant + '.gresource',
    command : [
      glib_compile_resources,
      '--sourcedir=@OUTDIR@',
      '--sourcedir=@CURRENT_SOURCE_DIR@',
      '--target=@OUTPUT@',
      '@INPUT@'
    ],
    depends : [gtk4_assets, gtk4_hidpi_assets, gtk4_stylesheets],
    depend_files : [gtk4_asset_files, gtk4_hidpi_asset_files],
    build_by_default : true
  )

  # install gresource
  meson.add_install_script(
    install_file,
    gresource.full_path(),
    '$MESON_INSTALL_DESTDIR_PREFIX' / install_dir.get(variant) / common_dirs.get('gtk4') / 'gtk.gresource'
  )

  # install gtk.css files

  gtk4_css = configure_file(
    capture : true,
    command : ['echo', '@import url("resource:///org/gnome/arc-theme/' + output_css + '");'],
    output : 'gtk-' + variant + '.css'
  )

  install_data(
    gtk4_css,
    install_dir : prefix / install_dir.get(variant) / common_dirs.get('gtk4'),
    rename : ['gtk.css']
  )

  #FIXME gtk-dark.css gets overwritten for subsequent variants, resulting in build warnings
  gtk4_dark_css = configure_file(
    capture : true,
    command : ['echo', '@import url("resource:///org/gnome/arc-theme/' + output_css_dark + '");'],
    output : 'gtk-dark.css',
    install : true,
    install_dir : prefix / install_dir.get(variant) / common_dirs.get('gtk4')
  )
endforeach
