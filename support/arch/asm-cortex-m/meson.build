# SPDX-FileCopyrightText: 2023 - 2024 Ledger SAS
# SPDX-License-Identifier: Apache-2.0

target_cpu = 'cortex-@0@'.format(kconfig_data.get_unquoted('CONFIG_ARCH_ARM_CORTEX_NAME'))
target_has_fpu = kconfig_data.get('CONFIG_HAS_FPU', 0) == 1
target_use_fpu = kconfig_data.get('CONFIG_FPU_SOFT_ABI', 0) != 1

c_compiler = meson.get_compiler('c')

target_rust = kconfig_data.get_unquoted('CONFIG_RUSTC_TARGET')
target_rustargs = [
    '-Ctarget-cpu='+ target_cpu,
    '-Clinker=' + c_compiler.cmd_array()[0],
]

# Implicit rustargs are ones set from meson cross file
# Those args do not need to be add to project args
# But need to be exported for rust sdk.
implicit_target_rustargs = []

if with_uapi_opt
    add_languages('rust', native: false, required: true)
    rust_compiler = meson.get_compiler('rust')

    # rustc target must be defined in cross file
    # check consistency between `.config` selected target and used cross-file
    rust_target_opt = '--target'
    target_rust_cross_file = ''

    # XXX:
    #  Only --target is handled here
    #  Other implicit args won't be exported.
    #  If needed, open a Issue on Github.
    foreach arg: rust_compiler.cmd_array()
        if arg.startswith(rust_target_opt)
            implicit_target_rustargs += [arg]
            target_rust_cross_file = arg.split('=')[1]
        endif
    endforeach

    if target_rust_cross_file == ''
        warning('rustc target is missing from cross file')
        target_rustargs += ['--target=' + target_rust]
    elif target_rust_cross_file != target_rust
        error(
            'rust target mismatch (@0@ / @1@) check .config and/or cross file'.format(
                target_rust,
                target_rust_cross_file
            )
        )
    endif
endif

target_cflags = [
    '-mcpu=' + target_cpu,
    kconfig_data.get_unquoted('CONFIG_ISA_CFLAGS'),
    kconfig_data.get_unquoted('CONFIG_ENDIANNESS_CFLAGS'),
    kconfig_data.get_unquoted('CONFIG_FPU_ABI_CFLAGS'),
]

# Set correct compiler flags for gcc and rustc according to FPU usage and float ABI
#
# For C, append FPU cflags if FPU used, this flags is set by kconfig.
# For Rust:
#  - kconfig set correct arch if hard float is required.
#  - As target-cpu is set, if target has cpu, it is used (ABI depends on target triple
#    w/ or w/o hf suffix. This is set by kconfig dependencies too.
#  - In the case of float software emulation on target w/ FPU, one has to add `-fpregs`
#    to `target-features` compile options
if target_has_fpu
if target_use_fpu
target_cflags += [kconfig_data.get_unquoted('CONFIG_FPU_CFLAGS')]
else
target_rustargs += [ '-Ctarget-feature=-fpregs' ]
endif
endif

foreach link_arg: target_link_args
    target_rustargs += ['-Clink-args=' + link_arg]
endforeach

target_arch_args += target_cflags

summary(
    {
        'cflags': target_cflags,
        'rustargs': target_rustargs,
        'implicit rustargs': implicit_target_rustargs,
    },
    bool_yn: true,
    section: 'Target specific compile args'
)
