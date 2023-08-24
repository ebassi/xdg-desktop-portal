gdbus_codegen = find_program('gdbus-codegen', native: true, required: get_option('documentation'))
sphinx_build = find_program('sphinx-build', native: true, required: get_option('documentation'))

if get_option('documentation').enabled() and gdbus_codegen.found() and sphinx_build.found()
  fs = import('fs')

  # Gather the XML files under data
  all_interfaces_xml = []
  foreach i: portal_sources
    all_interfaces_xml += i
  endforeach
  foreach i: portal_impl_sources
    all_interfaces_xml += i
  endforeach

  # Replace the extension for gdbus-codegen
  all_interfaces_rst = []
  foreach i: all_interfaces_xml
    f = fs.name(i)
    all_interfaces_rst += fs.replace_suffix(f, '.rst')
  endforeach

  interfaces_rst = custom_target('interfaces_rst',
    input: all_interfaces_xml,
    output: all_interfaces_rst,
    command: [ gdbus_codegen, '--generate-rst', 'doc/doc', '@INPUT@' ],
  )

  docs_sources = [
    'conf.py',

    'index.rst',
    'common-conventions.rst',
    'portals-conf.rst',
    'portal-interfaces.rst',
    'implementation-interfaces.rst',
  ]

  copied_docs_sources = []
  foreach d: docs_sources
    copied_docs_sources += fs.copyfile(d)
  endforeach

  custom_target('docs',
    input: copied_docs_sources + interfaces_rst,
    output: 'html',
    command: [ sphinx_build, meson.current_build_dir(), '@OUTPUT@' ],
    build_by_default: true,
  )
endif

if get_option('man-pages').enabled() and rst2man.found()
  rst2man_flags = [
    '--syntax-highlight=none',
  ]
  
  man_pages = [
    { 'input': 'portals-conf.rst', 'output': 'portals.conf', 'section': '5' },
  ]

  foreach man_page: man_pages
    man_input = man_page.get('input')
    man_output = man_page.get('output', man_input)
    man_section = man_page.get('section', '1')
    man_full = '@0@.@1@'.format(man_output, man_section)

    custom_target('man-' + man_output,
      input: man_input,
      output: man_full,
      command: [ rst2man, rst2man_flags, '@INPUT@' ],
      capture: true,
      install: true,
      install_dir: get_option('mandir') / 'man@0@'.format(man_section),
    )
  endforeach
endif
