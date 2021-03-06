project('mimic-english', 'c', default_options : ['c_std=c99'], version: '0')

build_plugin = get_option('plugin')
train_lex_lts = get_option('rebuild_lex_lts')

cmulex_src = [
  'src/cmulex/cmu_lex.c',
  'src/cmulex/cmu_postlex.c']

if train_lex_lts
  build_rules = custom_target('build_lex_lts',
                              command : '@0@/src/cmulex/en/en_make_lex'.format(meson.current_source_dir()),
                              depend_files : ['src/cmulex/en/cmu_allowables.json',
                                              'src/cmulex/en/cmudict.scm',
                                              'src/cmulex/en/cmulex.scm',
                                              'src/cmulex/en/cmu_phoneset.scm'],
                              output : ['cmu_lex_data.c', 'cmu_lex_entries.c', 'cmu_lts_rules.c'])

  cmulex_src += build_rules
  
else
  cmulex_src += [
    'src/cmulex/cmu_lex_data.c',
    'src/cmulex/cmu_lex_entries.c',
    'src/cmulex/cmu_lts_rules.c'
  ]
endif

usenglish_src = [
  'src/usenglish/us_aswd.c',
  'src/usenglish/us_dur_stats.c',
  'src/usenglish/us_durz_cart.c',
  'src/usenglish/usenglish.c',
  'src/usenglish/us_expand.c',
  'src/usenglish/us_f0lr.c',
  'src/usenglish/us_f0_model.c',
  'src/usenglish/us_ffeatures.c',
  'src/usenglish/us_gpos.c',
  'src/usenglish/us_int_accent_cart.c',
  'src/usenglish/us_int_tone_cart.c',
  'src/usenglish/us_nums_cart.c',
  'src/usenglish/us_phoneset.c',
  'src/usenglish/us_pos_cart.c',
  'src/usenglish/us_text.c']

sources = usenglish_src + cmulex_src

includes = [include_directories('include')]

# Core Dependency:
mimiccore = dependency('ttsmimic_core', required: false)
if mimiccore.found()
  mimic_plugin_dir = mimiccore.get_pkgconfig_variable('mimic_plugin_dir')
else
  # We compile it:
  mimic_core_sub = subproject('mimic-core')
  mimiccore = mimic_core_sub.get_variable('ttsmimic_core_dep')
  mimic_plugin_dir = mimic_core_sub.get_variable('mimic_plugin_dir')
endif

# If mimic-core does not have plugin support, disable the plugin
if mimic_plugin_dir == ''
    build_plugin = false
endif
message('Build mimic-english plugin: @0@'.format(build_plugin))


en_us_header_subdir = join_paths('ttsmimic', 'lang', 'en_US')

ttsmimic_english_lib = library('ttsmimic_english', sources,
                               include_directories : includes,
                               dependencies: mimiccore,
                               install: true)

ttsmimic_english_dep = declare_dependency(link_with : ttsmimic_english_lib, 
                                          include_directories : includes)


if build_plugin
  ttsmimic_english_plugin_lib = library(
    'ttsmimic_english_plugin',
    ['src/mimic_english_plugin.c'],
    include_directories : includes,
    dependencies: [mimiccore, ttsmimic_english_dep],
    install_dir: mimic_plugin_dir,
    install: true)

  ttsmimic_english_plugin_dep = declare_dependency(
    link_with : ttsmimic_english_plugin_lib, 
    include_directories : includes)
endif



############ Headers #################

ttsmimic_headers = ['include/cmu_lex.h', 'include/usenglish.h']

install_headers(ttsmimic_headers, subdir : en_us_header_subdir)

# FIXME: Some files were autogenerated from festival sources.
# It would be cool to regenerate them from sources easily.
#regexes:
#	./make_us_regexes $(BINDIR) >us_regexes.h
#
#us_pos_cart.c: 
#	$(ESTDIR)/../festival/bin/festival \
#           -b $(MIMICDIR)/tools/make_cart.scm \
#               $(MIMICDIR)/tools/make_pos.scm 
# src/cmulex/make_cmulex_helper.py
#  'src/cmulex/make_cmulex',
#  'src/usenglish/us_pos.tree',
#  'src/usenglish/us_regexes.h',
#  'src/usenglish/us_ffeatures.h',
#  'src/usenglish/us_f0.h',
#  'src/usenglish/us_int_tone_cart.h',
#  'src/usenglish/us_int_accent_cart.h',
#  'src/usenglish/us_phrasing_cart.h',
#  'src/usenglish/us_nums_cart.h',
#  'src/usenglish/us_durz_cart.h',
#  'src/usenglish/us_pos_cart.h',

########### pkg-config #############

pkg = import('pkgconfig')

pkg.generate(libraries : ttsmimic_english_lib,
             name : 'ttsmimic_english',
             subdirs: en_us_header_subdir,
             requires: 'ttsmimic_core',
             description: 'English support for text to speech synthesis in mimic.',
             version: meson.project_version())

