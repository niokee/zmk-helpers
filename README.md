# zmk-nodefree-config

ZMK allows users to customize their keyboard layout by providing a Devicetree file
(`.keymap`). The specific syntax requirements of the Devicetree file format can make
this process a bit daunting for new users. 

This repository provides simple convenience macros that simplify the configuration for
many common use cases. It results in a "node-free" user configuration with a more
streamlined
syntax.

## setup

1. Copy the file `helper.dtsi` from this repository into the `config` folder of your
   private `zmk-config` repository.
2. Source `helper.dtsi` near the top of your `.keymap` file:
    ```
    #include <behaviors.dtsi>
    #include <dt-bindings/zmk/keys.h>
    #include "helper.dtsi"
    ```

## usage

The file provides two convenience macros. `ZMK_BEHAVIOR` creates new behaviors while
`ZMK_LAYER` adds new layers to your keymap. See the [example.keymap
file](example.keymap) in this repository for a complete example.

### ZMK\_BEHAVIOR

The function is invoked by calling `ZMK_BEHAVIOR(name, type, specification)`, expecting
3 arguments:
* `name` is a unique string (e.g., `my_behavior`). It can later be used to refer to
    the new behavior by preceding it with an "`&`" (e.g., `&my_behavior`)
* `type` refers to the type of the new behavior. It must be one of the following:
  `caps_word`, `hold_tap`, `key_repeat`, `macro`, `mod_morph`, `sticky_key` or
  `tap_dance`. Note the use of underscores (`_`) instead of hyphens (`-`) here.
* `specification` contains the definition of the new behavior. It should contain the
  body of the corresponding [ZMK behavior configuration](https://zmk.dev/docs/config/behaviors)
  without the `label`, `#binding-cells` and `compatible` properties and without the
  surrounding node-specification.

#### Example 1: Creating a custom "homerow mod" tap-hold behavior

```
ZMK_BEHAVIOR(hrm, hold_tap,
    flavor = "balanced";
    tapping-term-ms = <280>;
    quick-tap-ms = <125>;
    global-quick-tap;
    bindings = <&kp>, <&kp>;
)
```

The new behavior can be added to the keymap-layout using `&hrm` (e.g., `&hrm LSHIFT T`
creates a key that yields `T` on tap and `LSHIFT` on hold, using the custom
configuration above).

#### Example 2: Creating a custom tap-dance key

```
ZMK_BEHAVIOR(ss_cw, tap_dance,
    tapping-term-ms = <200>;
    bindings = <&sk LSHFT>, <&caps_word>;
)
```

The new behavior can be added to the keymap-layout using `&ss_cw`. The key yields 
sticky-shift on tap and capsword on double tap;

#### Example 3: Creating a custom "win-sleep" macro

```
ZMK_BEHAVIOR(win_sleep, macro,
    wait-ms = <100>;
    tap-ms = <5>;
    bindings = <&kp LG(X) &kp U &kp S>;
)
```

This creates a "Windows sleep key" that can be added to the keymap-layout using
`&win_sleep`.

### ZMK\_LAYER

The function is invoked by calling `ZMK_LAYER(name, layout)`, expecting
2 arguments:
* `name` is a unique identifier string for the new layer (it isn't used elsewhere)
* `layout` provides the layout specification using the same syntax as the `bindings`
  property of the [ZMK keymap configuration](https://zmk.dev/docs/config/keymap)

Multiple layers can be defined by repeated calls of `ZMK_LAYER`. They will be ordered by
in the same order in which they are specified, with the first specified layer being the
"lowest" one ([see here for details](https://zmk.dev/docs/features/keymaps#layers)).

#### Example usage
```
ZMK_KEYMAP(default_layer,
     // ╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮   ╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮
          &kp Q         &kp W         &kp F         &kp P         &kp B             &kp J         &kp L         &kp U         &kp Y         &kp SQT
     // ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤   ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
          &hrm LGUI A   &hrm LALT R   &hrm LCTRL S  &hrm LSHFT T  &kp G             &kp M         &hrm RSHFT N  &hrm LCTRL E  &hrm LALT I   &hrm LGUI O
     // ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤   ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
          &kp Z         &kp X         &kp C         &kp D         &kp V             &kp K         &kp H         &kp COMMA     &kp DOT       &kp SEMI
     // ╰─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤   ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
                                      &kp ESC       &lt NAV SPACE &kp TAB           &kp RET       &ss_cw        &bs_del_num
     //                             ╰─────────────┴──── ────────┴─────────────╯   ╰─────────────┴─────────────┴─────────────╯
)

```
