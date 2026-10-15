[![pages-build-deployment](../../actions/workflows/pages/pages-build-deployment/badge.svg)](../../actions/workflows/pages/pages-build-deployment)

# tt-gds-action

[ULX3S](https://radiona.org/ulx3s/) GitHub action for hardening your [Tiny Tapeout](https://tinytapeout.com/) design into a manufacturable GDS file with FPGA tests.


## Branch Details

ULX3S support spans three coordinated repositories:

| Component | ULX3S development branch | Upstream proposal |
| --- | --- | --- |
| GitHub Action | [`ulx3s/tt-gds-action@experimental`](https://github.com/ulx3s/tt-gds-action/tree/experimental) | [TinyTapeout/tt-gds-action#47](https://github.com/TinyTapeout/tt-gds-action/pull/47) |
| FPGA build support | [`ulx3s/tt-support-tools@experimental`](https://github.com/ulx3s/tt-support-tools/tree/experimental) | [TinyTapeout/tt-support-tools#167](https://github.com/TinyTapeout/tt-support-tools/pull/167) |
| Verilog template/workflow | [`ulx3s/ttsky-verilog-template@ulx3s`](https://github.com/ulx3s/ttsky-verilog-template/tree/ulx3s) | [TinyTapeout/ttsky-verilog-template#23](https://github.com/TinyTapeout/ttsky-verilog-template/pull/23) |


The `main` branch of _this_ repository is a direct fork of the [TinyTapeout/tt-gds-action](https://github.com/TinyTapeout/tt-gds-action) and tracks upstream main.

The `main` ULX3S _template_ branch tracks the upstream [TinyTapeout/ttsky-verilog-template](https://github.com/TinyTapeout/ttsky-verilog-template).

The `ulx3s` [ttsky-verilog-template branch](https://github.com/ulx3s/ttsky-verilog-template/tree/ulx3s) contains template modifications to the template to support the ULX3S FPGA board
in [Pull Request #23](https://github.com/TinyTapeout/ttsky-verilog-template/pull/23) for direct integration from TinyTapeout:

```yaml
steps:
  # Not supported until merge of PR #23:
  TinyTapeout/tt-gds-action/fpga/ulx3s@ulx3s
```

Instead, use the `experimental` branch of this repository until the upstream PRs are merged. 


## Usage

Use the [ulx3s/ttsky-verilog-template](https://github.com/ulx3s/ttsky-verilog-template) as a starting point for your submission 
(or any of the other [TinyTapeout templates](https://github.com/orgs/TinyTapeout/repositories?q=template)).

A working minimal example is the [Hazard3-Doom tt-fpga-ulx workflow](https://github.com/ulx3s/Hazard3-Doom/blob/main/.github/workflows/tt-fpga-ulx.yaml).

Add a job step such as this to the workflow file:

```yaml
jobs:
  fpga-ulx3s:
    runs-on: ubuntu-24.04
    steps:
      - name: checkout repo
        uses: actions/checkout@v7
        with:
          submodules: recursive

      - name: FPGA bitstream for TT ASIC Sim (ULX3S ECP5)
        uses: ulx3s/tt-gds-action/fpga/ulx3s@experimental
        with:
          ecp5-device: 85k
          lpf: tt/fpga/ulx3s/ulx3s_v316.lpf
          artifact-name: fpga_ulx3s_ecp5
          uart-enabled: true
```

Where:

```text
owner
   |    repo name
   |      |        subdirectory (family container)
   |      |           |
   |      |           |   subdirectory (device sepcific action)  
   |      |           |    |    
   |      |           |    |   branch name
   |      |           |    |       |
ulx3s/tt-gds-action/fpga/ulx3s@experimental
```

Additional action parameters are specified in the `with:`:

```yaml
        with:
          ecp5-device: 85k
          lpf: tt/fpga/ulx3s/ulx3s_v20.lpf
          artifact-name: fpga_ulx3s_ecp5
          uart-enabled: true
```

The optional `oss-cad-suite-version` input selects the OSS CAD Suite release.
It defaults to `2026-07-20` for ULX3S, ULX4M, and iCE40 UP5K actions.
For example, to override the default in a calling workflow:

```yaml
        with:
          oss-cad-suite-version: '2026-07-20'
```

The ULX3S and ULX4M root action also forwards this input to the selected board action.
The setup step uses the calling workflow's GitHub token for authenticated release lookups; no separate secret is needed.

The `with: lpf:` constraint files are provided by [`ulx3s/tt-support-tools`](https://github.com/ulx3s/tt-support-tools/tree/experimental/fpga/ulx3s) and are checked out under `tt/fpga/ulx3s` by the action.

## Usage with GitHub Marketplace Syntax

As of v0.0.3, the action can be used with the [GitHub Marketplace](https://github.com/marketplace/actions/tiny-tapeout-gds-action-for-the-ulx3s-and-ulx4m-ecp5-fpga) syntax.
This is when pressing the "Use this action" button on the Marketplace page. The syntax is equivalent to the above, but with a different `uses:` path:

```yaml
      - name: Tiny Tapeout GDS Action for the ULX3S and ULX4M ECP5 FPGA
        # see https://github.com/marketplace/actions/tiny-tapeout-gds-action-for-the-ulx3s-and-ulx4m-ecp5-fpga
        uses: ulx3s/tt-gds-action@v0.0.3
```

[PR#5](https://github.com/ulx3s/tt-gds-action/pull/5) added a root level dispatch to appease marketplace default syntax.

The `experimental` name is still the preferred name for ULX3S actions until fully tested. The `v0.0.3` tag is a stable release created for the Marketplace.

Tested [here](https://github.com/gojimmypi/ttgf-UART-FSM-TRNG-Lab/actions/runs/34155592940/job/101846628123).

## RTL Structure

To effectively use with the TT toolchain, the project file should contain a module named `tt_[user]_[repo]` in the `src/project.v`. 
For [example this](https://github.com/gojimmypi/ttgf-UART-FSM-TRNG-Lab/blob/a59efdc8a51dcd5f2c868acfd4993b5c532e6dc0/src/project.v#L176) `tt_um_gojimmypi_ttsky_UART_FSM_TRNG_Lab`
which contains the standard TT interface from the [HDL templates](https://tinytapeout.com/hdl/templates/).

```verilog
#(
    /* Get project-wide params from project_config.v */
    parameter [31:0] CLOCK_HZ  = `PROJECT_CLOCK_HZ,    /* default clock is 25 MHz */
    parameter [31:0] UART_BAUD = `PROJECT_UART_BAUD    /* default UART is 115200 baud */
)
(
`ifdef ANALOG_ENABLED
    // Optional Analog
    //    input  wire       VGND,
    //    input  wire       VDPWR,    // 1.8v power supply
    //    input  wire       VAPWR,    // 3.3v power supply
`endif

    input  wire [7:0] ui_in,    // Dedicated inputs
    output wire [7:0] uo_out,   // Dedicated outputs
    input  wire [7:0] uio_in,   // IOs: Input path
    output wire [7:0] uio_out,  // IOs: Output path
    output wire [7:0] uio_oe,   // IOs: Enable path (active high: 0=input, 1=output)

    //    inout  wire [7:0] ua,       // Analog pins, only ua[5:0] can be used

    input  wire       ena,      // always 1 when the design is powered, so you can ignore it
    input  wire       clk,      // clock
    input  wire       rst_n     // reset_n - low to reset
);
```

Then the ULX3S FPGA test wrapper is typically placed in a _different_ directory with a _different_ module name, for example `top_ulx3s` in `ulx3s/fpga_ulx3s.v`, 
for [example](https://github.com/gojimmypi/ttgf-UART-FSM-TRNG-Lab/blob/main/ulx3s/top_ulx3s.v):

```verilog
module top_ulx3s (
    input  wire        clk_25mhz,
`ifdef ULX3S_BOARD_VERSION_v20
    /* The reference lpf file has an unconnected gn[12] */
    input  wire       \gn[12] ,
`elsif ULX3S_BOARD_VERSION_v307
    input  wire       \gn[12] ,
`else
    input  wire        gn12, /* contains optional 50 MHz clock, see ULX3S_USE_GN12_50MHZ */
`endif

    input  wire [6:0]  btn,
    output wire [7:0]  led,

    /* External PMOD-style UART pins. */
    input  wire        gp0,
    output wire        gp1,

    /* USB FTDI UART. */
    output wire        ftdi_rxd,
    input  wire        ftdi_txd,

/* Experimental RTS/DTR to control ESP32 boot mode during serial programming.
 * See also ESP32_BOOT_CONTROL_ENABLED, below. */
`ifdef ESP32_BOOT_RTS_DTR_ENABLED
    input  wire        ftdi_nrts,
    input  wire        ftdi_ndtr,
`endif /* ESP32_BOOT_RTS_DTR_ENABLED */
    /* ESP32 UART and boot control. */
    output wire        wifi_rxd,
    input  wire        wifi_txd,

    /* Optional ESP32 programming control signals. See esp32_prog_ctrl.v for details. */
`ifdef HAS_ESP32_PROG_CTRL
    output wire        wifi_en,
    output wire        wifi_gpio0,
`endif /* HAS_ESP32_PROG_CTRL */

    /* Keep board powered. */
    output wire        shutdown
); /* top_ulx3s input */

/*******************************************************************************
 * top module implementation
 ******************************************************************************/

    wire clk_ulx3s;

`ifdef ULX3S_BOARD_VERSION_v20
    /* The reference v20 lpf file contains BOTH gn12 AND gn[12]. To avoid complaints: */
    wire gn12;
    assign gn12 = \gn[12] ;
`elsif ULX3S_BOARD_VERSION_v307
    /* The reference v3.07 lpf file contains BOTH gn12 AND gn[12]. (used v20 LPF): */
    wire gn12;
    assign gn12 = \gn[12] ;
`endif /* ULX3S_BOARD_VERSION_v20 */

`ifdef ULX3S_USE_GN12_50MHZ
    /* Reminder this 50 MHz clock ONLY comes from HDMI */
    assign clk_ulx3s = gn12;
`else
    assign clk_ulx3s = clk_25mhz;
`endif

    wire [7:0] ui_in;
    wire [7:0] uio_in;
    wire [7:0] uo_out;
    wire [7:0] uio_out;
    wire [7:0] uio_oe;

    wire rst_n;
    wire ena;

... etc
```

## Build flow and artifacts

The composite action performs these steps:

1. Checks out `tt-support-tools` into `tt/`.
2. Installs the required Python dependencies and OSS CAD Suite.
3. Reads the Tiny Tapeout project configuration.
4. Creates the ULX3S FPGA wrapper using the project's `top_module`.
5. Synthesizes the design with Yosys and `synth_ecp5`.
6. Runs nextpnr-ecp5 using the selected device and LPF file.
7. Packs the ECP5 configuration into a `.bit` file with `ecppack`.
8. Uploads the build outputs as a GitHub Actions artifact.

With the defaults above, the uploaded artifact is named:

```text
fpga_ulx3s_ecp5_85k
```

## Related work

The upstream ULX3S integration is tracked by:

- [TinyTapeout/ttsky-verilog-template issue #22](https://github.com/TinyTapeout/ttsky-verilog-template/issues/22)
- [TinyTapeout/tt-support-tools PR #167](https://github.com/TinyTapeout/tt-support-tools/pull/167)
- [TinyTapeout/tt-gds-action PR #47](https://github.com/TinyTapeout/tt-gds-action/pull/47)
- [TinyTapeout/ttsky-verilog-template PR #23](https://github.com/TinyTapeout/ttsky-verilog-template/pull/23)


## Support

### Discord channel

- [https://discord.gg/qwMUk6W](https://discord.gg/qwMUk6W) (problems/question/general chat); [#tinytapeout](https://discord.com/channels/690209441953480758/1521208647651426435)

### Gitter channel

- [https://gitter.im/ulx3s/Lobby](https://gitter.im/ulx3s/Lobby) (Focused on development)

### Email

- [ulx3s.fpga@gmail.com](mailto:ulx3s.fpga@gmail.com) (If you do not use chats)

## Updating the action

To update the release tag of the action simply add a new release tag to this repository.

To update an existing `@` version, in this case `experimental`, simply add additional commits.

Once fully tested with no anomalies confirmed, a `v1` branch will be added: `ulx3s@v1`

## License

Copyright 2023-2025 Tiny Tapeout LTD

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
