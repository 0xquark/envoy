load("//bazel:android_debug_info.bzl", "android_debug_info")
load("@envoy//bazel:envoy_build_system.bzl", "envoy_package")

licenses(["notice"])  # Apache 2

envoy_package()

# Support for JNI, using NDK on Android and Java otherwise, since the JNI APIs
# differ a bit between the two.
cc_library(
    name = "jni_support_lib",
    srcs = select({
        "@envoy//bazel:android": ["ndk_jni_support.cc"],
        "//conditions:default": ["java_jni_support.cc"],
    }),
    hdrs = ["jni_support.h"],
    linkopts = select({
        "@envoy//bazel:android": ["-llog"],
        "//conditions:default": [],
    }),
    deps = [
        "//library/common/jni/import:jni_import_lib",
    ],
)

# Various JNI helper libraries.
cc_library(
    name = "jni_utility_lib",
    srcs = [
        "jni_utility.cc",
        "jni_version.cc",
    ],
    hdrs = [
        "jni_utility.h",
        "jni_version.h",
    ],
    deps = [
        ":jni_support_lib",
        "//library/common/jni/import:jni_import_lib",
        "//library/common/types:c_types_lib",
        "//library/common/types:managed_types_lib",
        "@envoy//source/common/common:assert_lib",
    ],
)

# Implementations of the various "native" Java methods for classes
# in library/java/io/envoyproxy/envoymobile.
# TODO(RyanTheOptimist): Is there a better name for this? I'm not sure what
#     "interface" refers to in this file/lib. Maybe we should have a convention
#     where the name of the java classes are part of the file name?
cc_library(
    name = "jni_interface_lib",
    srcs = [
        "jni_interface.cc",
    ],
    deps = [
        ":android_network_utility_lib",
        ":jni_utility_lib",
        "//library/common:envoy_main_interface_lib",
        "//library/common/api:c_types",
        "//library/common/extensions/cert_validator/platform_bridge:c_types_lib",
        "//library/common/extensions/key_value/platform:config",
        "//library/common/types:managed_types_lib",
    ],
    # We need this to ensure that we link this into the .so even though there are no code references.
    alwayslink = True,
)

# Cert verification related functions which call into AndroidNetworkLibrary.
cc_library(
    name = "android_network_utility_lib",
    srcs = [
        "android_network_utility.cc",
    ],
    hdrs = [
        "android_network_utility.h",
    ],
    deps = [
        ":jni_utility_lib",
        "//library/common/api:c_types",
        "//library/common/data:utility_lib",
        "//library/common/extensions/cert_validator/platform_bridge:c_types_lib",
        "//library/common/jni/import:jni_import_lib",
        "//library/common/types:c_types_lib",
        "@envoy//bazel:boringssl",
    ],
)

# Implementations of the "native" Java methods for AndroidJniLibrary.java
cc_library(
    name = "android_jni_interface_lib",
    srcs = [
        "android_jni_interface.cc",
    ],
    deps = [
        ":android_network_utility_lib",
        ":jni_interface_lib",
        ":jni_support_lib",
        ":jni_utility_lib",
        "//library/common:envoy_main_interface_lib",
        "//library/common/jni/import:jni_import_lib",
    ],
    # We need this to ensure that we link this into the .so even though there are no code references.
    alwayslink = True,
)

# Library which contains all the JNI related targets.
cc_library(
    name = "envoy_jni_lib",
    linkopts = select({
        "@envoy//bazel:dbg_build": ["-Wl,--build-id=sha1"],
        "//conditions:default": [],
    }),
    deps = [
        ":android_jni_interface_lib",
        ":android_network_utility_lib",
        ":jni_interface_lib",
        ":jni_support_lib",
        ":jni_utility_lib",
    ],
    # We need this to ensure that we link this into the .so even though there are no code references.
    alwayslink = True,
)

config_setting(
    name = "android_armeabi",
    constraint_values = [
        "@platforms//cpu:armv7",
        "@platforms//os:android",
    ],
)

# Main dynamic library for the Envoy Mobile aar
cc_binary(
    name = "libenvoy_jni.so",
    linkopts = select({
        "@envoy//bazel:dbg_build": ["-Wl,--build-id=sha1"],
        "//conditions:default": [],
    }),
    linkshared = True,
    deps = [
        ":envoy_jni_lib",
        "@envoy_mobile_extra_jni_deps//:extra_jni_dep",
    ],
)

android_debug_info(
    name = "libenvoy_jni.so.debug_info",
    dep = "libenvoy_jni.so",
)

cc_library(
    name = "android_jni_utility_lib",
    srcs = [
        "android_jni_utility.cc",
    ],
    hdrs = [
        "android_jni_utility.h",
    ],
    deps = [
        "@envoy//source/common/common:assert_lib",
        "//library/common/types:c_types_lib",
    ] + select({
        "@envoy//bazel:android": [
            ":jni_support_lib",
            ":jni_utility_lib",
            "//library/common/data:utility_lib",
            "//library/common/jni/import:jni_import_lib",
        ],
        "//conditions:default": [],
    }),
)
