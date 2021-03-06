# Copyright Vertex.AI.

package(default_visibility = ["//visibility:public"])

load(
    "//bzl:plaidml.bzl",
    "plaidml_cc_binary",
    "plaidml_cc_library",
    "plaidml_cc_test",
    "plaidml_proto_library",
    "plaidml_py_binary",
    "plaidml_py_library",
    "plaidml_py_test",
    "plaidml_py_wheel",
    "plaidml_cc_version",
)
load("@bazel_tools//tools/build_defs/pkg:pkg.bzl", "pkg_tar", "pkg_deb")

exports_files(["requirements.txt"])

plaidml_proto_library(
    name = "proto",
    srcs = ["plaidml.proto"],
)

plaidml_cc_version(
    name = "version",
    prefix = "PLAIDML",
)

plaidml_cc_library(
    name = "api",
    srcs = [
        "plaidml.cc",
        ":version",
    ],
    hdrs = [
        "plaidml.h",
        "plaidml++.h",
    ],
    deps = [
        ":proto_cc",
        "//base/config",
        "//base/eventing/file",
        "//base/util:runfiles_db",
        "//plaidml/base",
        "//tile/base",
        "//tile/base:program_cache",
        "//tile/hal/opencl",
        "//tile/platform/local_machine",
        "//tile/proto:proto_cc",
        "@half_repo//:half",
        "@minizip_repo//:minizip",
    ] + select({
        "@toolchain//:windows_x86_64": [],
        "@toolchain//:linux_arm_32v7": [],
        "@toolchain//:linux_arm_64v8": [],
        "//conditions:default": ["//tile/hal/cpu"],
    }),
)

# The PLAIDML C library, defined as a Bazel dynamically-loaded library.
# This allows the library to be used as a data dependency in Bazel rules.
plaidml_cc_binary(
    name = "libplaidml.so",
    srcs = select({
        "@toolchain//:windows_x86_64": ["dummy.cc"],  # Work around for https://github.com/bazelbuild/bazel/issues/4003
        "//conditions:default": [],
    }),
    linkshared = 1,
    deps = [
        ":api",
    ],
)

plaidml_cc_binary(
    name = "plaidml.dll",
    srcs = select({
        "@toolchain//:windows_x86_64": ["dummy.cc"],  # Work around for https://github.com/bazelbuild/bazel/issues/4003
        "//conditions:default": [],
    }),
    linkshared = 1,
    deps = [
        ":api",
    ],
)

plaidml_cc_library(
    name = "api_cc_deps",
    visibility = ["//visibility:private"],
    deps = [
        ":proto_cc",
        "//tile/base",
        "//tile/base:program_cache",
        "//tile/proto:proto_cc",
    ],
)

plaidml_cc_test(
    name = "plaidml_test",
    srcs = ["plaidml_test.cc"],
    deps = [
        ":api",
        "//testing:matchers",
        "//testing:plaidml_config",
    ],
)

plaidml_cc_test(
    name = "dtype_vec_test",
    srcs = ["dtype_vec_test.cc"],
    deps = [
        ":api",
        "//plaidml:proto_cc",
        "//testing:matchers",
        "//testing:plaidml_config",
        "//tile/hal/opencl:proto_cc",
        "//tile/platform/local_machine:proto_cc",
        "@half_repo//:half",
    ],
)

plaidml_cc_test(
    name = "matmul_fuzz_test",
    timeout = "eternal",
    srcs = ["matmul_fuzz_test.cc"],
    deps = [
        ":api",
        "//testing:plaidml_config",
    ],
)

plaidml_cc_test(
    name = "uint8_test",
    srcs = ["uint8_test.cc"],
    deps = [
        ":api",
        "//testing:matchers",
        "//testing:plaidml_config",
    ],
)

plaidml_cc_test(
    name = "cpp_test",
    srcs = ["cpp_test.cc"],
    deps = [
        ":api",
        "//testing:plaidml_config",
    ],
)

plaidml_cc_test(
    name = "network_test",
    size = "large",
    srcs = ["network_test.cc"],
    copts = select({
        "//bzl:with_callgrind": ["-DWITH_CALLGRIND"],
        "//conditions:default": [],
    }),
    data = [
        "testdata/resnet50.tpb",
        "testdata/xception.tpb",
    ],
    tags = ["manual"],
    deps = [
        ":api",
        "//testing:plaidml_config",
    ],
)

filegroup(
    name = "configs",
    srcs = [
        "config.json",
        ":experimental.json",
    ],
)

genrule(
    name = "experimental_json",
    srcs = select({
        "@toolchain//:windows_x86_64": ["experimental.no_llvm.json"],
        "@toolchain//:linux_arm_32v7": ["experimental.no_llvm.json"],
        "@toolchain//:linux_arm_64v8": ["experimental.no_llvm.json"],
        "//conditions:default": ["experimental.llvm.json"],
    }),
    outs = ["experimental.json"],
    cmd = "cp $< $@",
)

plaidml_py_binary(
    name = "setup",
    srcs = ["plaidml_setup.py"],
    main = "plaidml_setup.py",
    deps = [":py"],
)

plaidml_py_library(
    name = "py",
    srcs = [
        "__init__.py",
        "context.py",
        "exceptions.py",
        "library.py",
        "op.py",
        "plaidml_setup.py",
        "settings.py",
        "tile.py",
    ],
    data = [":configs"] + select({
        "@toolchain//:windows_x86_64": [":plaidml.dll"],
        "//conditions:default": [":libplaidml.so"],
    }),
)

plaidml_py_test(
    name = "settings_test",
    srcs = ["settings_test.py"],
    deps = [
        ":py",
    ],
)

plaidml_py_test(
    name = "py_plaidml_test",
    srcs = ["plaidml_test.py"],
    main = "plaidml_test.py",
    deps = [
        ":py",
        "//testing:plaidml_py",
    ],
)

plaidml_py_test(
    name = "py_tile_test",
    srcs = ["tile_test.py"],
    main = "tile_test.py",
    deps = [
        ":py",
        "//testing:plaidml_py",
    ],
)

pkg_tar(
    name = "pkg",
    srcs = glob([
        "**/*",
    ]),
    package_dir = "plaidml",
    strip_prefix = ".",
)

pkg_tar(
    name = "py_deb_data",
    srcs = [":py"],
    package_dir = "/usr/lib/python2.7/dist-packages/plaidml",
)

pkg_deb(
    name = "python-plaidml",
    data = ":py_deb_data",
    depends = [
        "python",
    ],
    description = "Vertex.AI PlaidML Python",
    homepage = "https://www.vertex.ai",
    maintainer = "eng@vertex.ai",
    package = "python-plaidml",
    priority = "extra",
    section = "non-free/misc",
    tags = ["deb"],
    version = "0.0.1",
)

plaidml_py_wheel(
    name = "wheel",
    package_name = "plaidml",
    srcs = [
        ":py",
        ":configs",
    ] + select({
        "@toolchain//:windows_x86_64": ["//plaidml:plaidml.dll"],
        "//conditions:default": ["//plaidml:libplaidml.so"],
    }),
    config = ":setup.cfg",
    package_prefix = "plaidml",
    platform = select({
        "@toolchain//:linux_arm_32v7": "linux_armv7l",
        "@toolchain//:macos_x86_64": "macosx_10_10_x86_64",
        "@toolchain//:windows_x86_64": "win_amd64",
        "//conditions:default": "manylinux1_x86_64",
    }),
    python = "py2.py3",
)

# HACK: pex doesn't yet support manylinux wheels.
plaidml_py_wheel(
    name = "wheel_linux_x86_64",
    package_name = "plaidml",
    srcs = [
        ":configs",
        ":py",
        "//plaidml:libplaidml.so",
    ],
    config = ":setup.cfg",
    package_prefix = "plaidml",
    platform = "linux_x86_64",
    python = "py2.py3",
)
