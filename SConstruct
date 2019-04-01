import os
import subprocess
import shlex
import pulp_config as plpconfig
import SCons.Util

install_dir = os.environ.get('INSTALL_DIR')
target_install_dir = os.environ.get('TARGET_INSTALL_DIR')

if install_dir is None:
  install_dir = 'install'

if target_install_dir is None:
  target_install_dir = 'install'

files = [ 'archi/pulp_defs.h', 'archi/pulp.h', 'archi/utils.h' ]
files.append('archi/gvsoc/gvsoc.h')

try:
  files += subprocess.check_output(shlex.split('plpfiles copy --item=archi_files')).decode('UTF-8').split()
except subprocess.CalledProcessError as e:
  print (e.output)
  raise

configs = plpconfig.get_configs_from_env()

def append_file(file):
  global files
  if not file in files:
    files.append(file)


for config in configs:

  # The old system is storing the path to archi files in the json file
  # This has to be migrated so that only IP information is stored in the
  # json file and this build system is them copying the archi files according
  # to the IP information found in the json file.

  chip = config.get('**/chip/pulp_chip_family').get()

  # UDMA I2S
  udma_i2s = config.get_child_int('**/udma/i2s/version')
  if udma_i2s is not None:
    append_file('archi/udma/i2s/udma_i2s_v%d.h' % udma_i2s)
    if udma_i2s == 1:
      append_file('archi/udma/i2s/udma_i2s_v%d_new.h' % udma_i2s)

  udma_hyper = config.get_child_int('**/udma/hyper/version')
  if udma_hyper is not None:
    append_file('archi/udma/hyper/udma_hyper_v%d.h' % udma_hyper)


  # UDMA MRAM
  udma_mram = config.get_child_int('**/udma/mram/version')
  if udma_mram is not None:
    append_file('archi/udma/mram/udma_mram_v%d.h' % udma_mram)


  # RTC
  rtc = config.get('**/soc/rtc')
  if rtc is None:
    rtc = config.get('**/chip/rtc')
  if rtc is not None:
    rtc_version = config.get_child_int('**/rtc/version')
    if rtc_version == 1 or rtc_version is None:
      append_file('archi/vendors/dolphin/rtc.h')
    else:
      append_file('archi/rtc/rtc_v%d.h' % (rtc_version))
  
  # PMU
  pmu = config.get_child_int('**/soc/pmu/version')
  if pmu is None:
    pmu = config.get_child_int('**/chip/pmu/version')

  if pmu is not None:
    append_file('archi/maestro/maestro_v1_new.h')
    if pmu >= 3:
      append_file('archi/maestro/maestro_v%d.h' % pmu)
    else:
      append_file('archi/maestro/maestro_v%d.h' % pmu)
      append_file('archi/maestro/maestro_v%d_new.h' % pmu)

  # ITC
  itc = config.get_child_int('**/soc/fc_itc/version')
  if itc is not None:
    append_file('archi/itc/itc_v%d.h' % itc)

  # GPIO
  gpio = config.get_child_int('**/gpio/version')
  if gpio is not None and gpio == 2:
    append_file('archi/gpio/gpio_v%d.h' % gpio)
    append_file('archi/gpio/gpio_v%d_new.h' % gpio)

  # TIMER
  timer = config.get_child_int('**/timer/version')
  if timer is not None:
    append_file('archi/timer/timer_v%d.h' % timer)


  # Chip specific files can be included here
  if chip == 'vega':
    append_file('archi/pwm/pwm_v1.h')
    append_file('archi/chips/vega/apb_soc_ctrl.h')
    append_file('archi/chips/vega/pmu.h')
  elif chip == 'pulpissimo':
    append_file('archi/chips/pulpissimo/apb_soc_ctrl.h')
  elif chip == 'gap':
    append_file('archi/pwm/pwm_v1.h')
  elif chip == 'wolfe':
    append_file('archi/pwm/pwm_v1.h')
    append_file('archi/chips/wolfe/pmu.h')
    append_file('archi/chips/wolfe/apb_soc.h')
    append_file('archi/chips/wolfe/apb_soc_ctrl_new.h')
  elif chip == 'wolfe':
    append_file('archi/pwm/pwm_v1.h')
  elif chip == 'vivosoc3':
    append_file('archi/chips/vivosoc3/fll.h')
    append_file('archi/chips/vivosoc3/freq.h')
  elif chip == 'vivosoc3_5':
    append_file('archi/chips/vivosoc3_5/fll.h')
    append_file('archi/chips/vivosoc3_5/freq.h')
  elif chip == 'vivosoc3_1':
    append_file('archi/chips/vivosoc3_1/fll.h')
    append_file('archi/chips/vivosoc3_1/freq.h')


  if chip == 'vega':
    out_file_path = 'archi/chips/%s/memory_map.h' % chip

    try:
        os.makedirs(os.path.dirname(out_file_path))
    except:
        pass

    with open('include/' + out_file_path, 'w') as out_file:
      with open('include/archi/chips/%s/memory_map.h.in' % chip, 'r') as in_file:
        content = in_file.read()

        content = content.replace('@l2_shared_size@', config.get_int('**/l2_shared/size'))

        out_file.write(content)

        append_file(out_file_path)




targets = []

for file in files:
  file_path = os.path.join('include', file)
  targets.append(InstallAs(os.path.join(install_dir, file_path), file_path))
  targets.append(InstallAs(os.path.join(target_install_dir, file_path), file_path))


Default(targets)
