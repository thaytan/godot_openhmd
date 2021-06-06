#!python
import os, subprocess

# Local dependency paths, adapt them to your setup
godot_glad_path = ARGUMENTS.get("glad", "glad")
godot_headers_path = ARGUMENTS.get("headers", "godot_headers/")
libusb_path = ARGUMENTS.get("libusb", os.getenv("LIBUSB_PATH", "libusb/"))
hidapi_path = ARGUMENTS.get("hidapi", os.getenv("HIDAPI_PATH", "hidapi/"))
openhmd_path = ARGUMENTS.get("openhmd", os.getenv("OPENHMD_PATH", "OpenHMD/"))

target = ARGUMENTS.get("target", "debug")

# platform= makes it in line with Godots scons file, keeping p for backwards compatibility
platform = ARGUMENTS.get("p", "linux")
platform = ARGUMENTS.get("platform", platform)

# destination path
godot_openhmd_path = 'demo/addons/godot-openhmd/bin/'

# This makes sure to keep the session environment variables on windows,
# that way you can run scons in a vs 2017 prompt and it will find all the required tools
env = Environment()
if platform == "windows":
    env = Environment(ENV = os.environ)

if ARGUMENTS.get("use_llvm", "no") == "yes":
    env["CXX"] = "clang++"

def add_sources(sources, directory):
    for file in os.listdir(directory):
        if file.endswith('.c'):
            sources.append(directory + '/' + file)
        elif file.endswith('.cpp'):
            sources.append(directory + '/' + file)

sources = []

platform_dir = ''
if platform == "osx":
    platform_dir = 'osx'
    godot_openhmd_path = godot_openhmd_path + 'osx/'
    env.Append(CCFLAGS = ['-g','-O3', '-arch', 'x86_64'])
    env.Append(LINKFLAGS = ['-arch', 'x86_64'])
    env.Append(LINKFLAGS=['-framework', 'Cocoa', '-framework', 'OpenGL', '-framework', 'IOKit'])
    env.Append(LIBS=['pthread'])

if platform == "linux":
    platform_dir = 'linux'
    godot_openhmd_path = godot_openhmd_path + 'linux/'
    env.Append(CCFLAGS = ['-fPIC', '-g','-O3'])
    env.Append(CXXFLAGS='-std=c++0x')
    env.Append(LINKFLAGS = ['-Wl,-R,\'$$ORIGIN\''])

if platform == "windows":
    platform_dir = 'win'
    godot_openhmd_path = godot_openhmd_path + 'win64/'
    if target == "debug":
        env.Append(CCFLAGS = ['-EHsc', '-D_DEBUG', '/MDd'])
    else:
        env.Append(CCFLAGS = ['-O2', '-EHsc', '-DNDEBUG', '/MD'])
    env.Append(LIBS=["opengl32", "setupapi", "advapi32.lib"])

####################################################################################################################################
# Link in glad
sources.append(godot_glad_path + "/glad.c")

####################################################################################################################################
# Build internal libusb on linux
if platform == 'linux':
    env.Append(CPPPATH=["libusb_include"]) # Location of the pre-generated libusb config.h for the internal build
    env.Append(CPPPATH=[libusb_path])
    env.Append(CPPPATH=[libusb_path + "libusb"])
    env.Append(CPPPATH=[libusb_path + "libusb/os"])

    libusb_sources = [
        "core.c",
        "descriptor.c",
        "hotplug.c",
        "io.c",
        "strerror.c",
        "sync.c"
    ]

    sources.append([libusb_path + "libusb/" + file for file in libusb_sources])

    sources.append(libusb_path + "libusb/os/linux_usbfs.c")
    sources.append(libusb_path + "libusb/os/linux_udev.c")
    sources.append(libusb_path + "libusb/os/events_posix.c")
    sources.append(libusb_path + "libusb/os/threads_posix.c")
    env.Append(CPPDEFINES=["OS_LINUX", "USE_UDEV", "HAVE_LIBUDEV"])
    env.Append(LIBS = ['udev'])

####################################################################################################################################
# Link in hidapi
hidapi_headers = hidapi_path + "hidapi/"
env.Append(CPPPATH=[hidapi_headers])

if platform == 'windows':
    sources.append(hidapi_path + "windows/hid.c" )
elif platform == 'linux':
    # If we can use the libusb version it should allow us to undo our detect.py changes
    # See thirdparty/hidapi/linux/README.txt and thirdparty/hidapi/udev/99-hid-rules for more info
    # env_openhmd.add_source_files(env.modules_sources, [ "#thirdparty/hidapi/linux/hid.c" ])
    sources.append(hidapi_path + "libusb/hid.c")
elif platform == 'osx':
    sources.append(hidapi_path + "mac/hid.c")

####################################################################################################################################
# Link in openhmd, we're linking in static.
env.Append(CFLAGS=["-DOHMD_STATIC"])
env.Append(CPPFLAGS=["-DOHMD_STATIC"])

# We don't include android because we're not compiling this for android at this time.
# Our native mobile VR class already handles android.
# We do include the Vive so we have basic native support even though we have an OpenVR implementation
env.Append(CFLAGS=["-DDRIVER_XGVR"])
#env.Append(CFLAGS=["-DDRIVER_ANDROID"])
env.Append(CFLAGS=["-DDRIVER_DEEPOON"])
#env.Append(CFLAGS=["-DDRIVER_EXTERNAL"])
env.Append(CFLAGS=["-DDRIVER_HTC_VIVE"])
env.Append(CFLAGS=["-DDRIVER_NOLO"])
env.Append(CFLAGS=["-DDRIVER_OCULUS_RIFT"])
env.Append(CFLAGS=["-DDRIVER_OCULUS_RIFT_S"])
env.Append(CFLAGS=["-DDRIVER_PSVR"])
env.Append(CFLAGS=["-DDRIVER_WMR"])
env.Append(CFLAGS=["-DDRIVER_VRTEK"])

env.Append(CPPFLAGS=["-DHAVE_OPENCV=1"])
env.ParseConfig('pkg-config --cflags --libs opencv')

openhmd_headers = openhmd_path + "include/"
env.Append(CPPPATH=[openhmd_headers])

openhmd_sources = [
    "fusion.c",
    "omath.c",
    "matrices.c",
    "openhmd.c",
    "shaders.c",
    "ukf.c",
    "unscented.c",
    "drv_3glasses/packet.c",
    "drv_3glasses/xgvr.c",
#    "drv_android/android.c",
    "drv_deepoon/deepoon.c",
    "drv_deepoon/packet.c",
    "drv_dummy/dummy.c",
#    "drv_external/external.c",
    "drv_htc_vive/packet.c",
    "drv_htc_vive/vive.c",
    "drv_nolo/nolo.c",
    "drv_nolo/packet.c",
    "drv_oculus_rift/packet.c",
    "drv_oculus_rift/rift-hmd-radio.c",
    "drv_oculus_rift/rift.c",
    "drv_oculus_rift/correspondence_search.c",
    "drv_oculus_rift/rift-sensor.c",
    "drv_oculus_rift/rift-sensor-mt9v034.c",
    "drv_oculus_rift/rift-sensor-opencv.cpp",
    "drv_oculus_rift/led_search.c",
    "drv_oculus_rift/rift-sensor-esp570.c",
    "drv_oculus_rift/rift-sensor-pose-helper.c",
    "drv_oculus_rift/rift-kalman-6dof.c",
    "drv_oculus_rift/rift-sensor-esp770u.c",
    "drv_oculus_rift/rift-sensor-uvc.c",
    "drv_oculus_rift/rift-sensor-ar0134.c",
    "drv_oculus_rift/rift-sensor-flicker.c",
    "drv_oculus_rift/rift-tracker.c",
    "drv_oculus_rift/rift-sensor-blobwatch.c",
    "drv_oculus_rift/rift-sensor-maths.c",
    "drv_oculus_rift_s/rift-s-controller.c",
    "drv_oculus_rift_s/rift-s-firmware.c",
    "drv_oculus_rift_s/rift-s-protocol.c",
    "drv_oculus_rift_s/rift-s-radio.c",
    "drv_oculus_rift_s/rift-s.c",
    "drv_psvr/packet.c",
    "drv_psvr/psvr.c",
    "drv_vrtek/packet.c",
    "drv_vrtek/vrtek.c",
    "drv_wmr/wmr.c",
    "drv_wmr/packet.c",
#    "ext_deps/miniz.c",
    "ext_deps/nxjson.c",
]

sources.append([openhmd_path + "src/" + file for file in openhmd_sources])

if platform == 'windows':
    sources.append(openhmd_path + "src/platform-win32.c" )
else:
    sources.append(openhmd_path + "src/platform-posix.c" )

####################################################################################################################################
# and add our main project
env.Append(CPPPATH=['.', godot_headers_path, godot_glad_path])

add_sources(sources, "src")

library = env.SharedLibrary(target=godot_openhmd_path + 'godot_openhmd', source=sources)
Default(library)
