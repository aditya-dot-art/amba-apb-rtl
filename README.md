# amba-apb-rtl
RTL design and verification of the AMBA APB (Advanced Peripheral Bus) protocol using Verilog/SystemVerilog, including APB master, slave, and protocol-compliant transaction handling

## APB Protocol

AMBA APB (Advanced Peripheral Bus) is a low-bandwidth, low-complexity
peripheral communication protocol defined as part of the ARM AMBA
architecture. It is intended for connecting low-speed and low-power
peripherals to a system interconnect.

### Key Characteristics

- Simple and low-complexity interface
- Low power and low bandwidth
- Non-pipelined protocol
- No burst transfers
- Supports read and write transactions
- Uses a two-phase transfer mechanism:
  - Setup phase
  - Access phase
- Supports peripheral transfer completion using `PREADY`
- Supports error reporting using `PSLVERR`

## APB Interface Signals

The AMBA APB interface consists of clock, reset, address, control, write-data, read-data, response, and optional user-defined signals. The Requester initiates transfers, while the Completer responds to the transactions. Signal widths are defined either as fixed values or configurable interface properties.

Signal Description Table

| Signal | Source | Width | Description |
|---|---|---:|---|
| `PCLK` | Clock | 1 | System clock. All APB transfers are synchronized to the rising edge of PCLK. |
| `PRESETn` | System Bus Reset | 1 | Active-low reset signal for the APB interface. |
| `PADDR` | Requester | ADDR_WIDTH | Address bus used for both read and write transfers. Supports up to 32-bit addressing. |
| `PPROT` | Requester | 3 | Protection attribute indicating privilege, security, and instruction/data access type. |
| `PSELx` | Requester | 1 | Select signal used to choose a peripheral for data transfer. |
| `PENABLE` | Requester | 1 | Indicates the Access phase of an APB transfer. |
| `PWRITE` | Requester | 1 | Transfer direction: HIGH for write and LOW for read. |
| `PWDATA` | Requester | DATA_WIDTH | Write data bus carrying data from the Requester to the selected peripheral. |
| `PSTRB` | Requester | DATA_WIDTH/8 | Write byte strobes indicating which byte lanes contain valid write data. |
| `PREADY` | Completer | 1 | Ready signal used to complete or extend an APB transfer with wait states. |
| `PRDATA` | Completer | DATA_WIDTH | Read data bus carrying data from the peripheral to the Requester. |
| `PSLVERR` | Completer | 1 | Optional error response signal indicating a failed APB transfer. |
| `PWAKEUP` | Requester | 1 | Optional wake-up signal indicating activity on the APB interface. |
| `PAUSER` | Requester | USER_REQ_WIDTH | Optional user-defined request attribute signal. |
| `PWUSER` | Requester | USER_DATA_WIDTH | Optional user-defined write-data attribute signal. |
| `PRUSER` | Completer | USER_DATA_WIDTH | Optional user-defined read-data attribute signal. |
| `PBUSER` | Completer | USER_RESP_WIDTH | Optional user-defined response attribute signal. |

## APB Write Transfer

An APB write transfer is used to transfer data from the Requester to the selected Completer. A write transfer consists of two main phases:

1. Setup Phase
2. Access Phase

### 1. Write Transfer Without Wait States

In a write transfer without wait states, the transfer completes in the minimum number of cycles.

#### Setup Phase

During the Setup phase:

- `PSEL` is asserted HIGH.
- `PENABLE` is LOW.
- `PWRITE` is HIGH, indicating a write transfer.
- `PADDR` contains the address of the peripheral register.
- `PWDATA` contains the data to be written.

The `PADDR`, `PWRITE`, and `PWDATA` signals must be valid when `PSEL` is asserted.

#### Access Phase

In the next clock cycle:

- `PSEL` remains HIGH.
- `PENABLE` is asserted HIGH.
- `PADDR`, `PWRITE`, and `PWDATA` remain stable.
- The Completer asserts `PREADY` HIGH.

When `PREADY` is HIGH during the Access phase, the write transfer completes at the rising edge of `PCLK`.

After the transfer completes:

- `PENABLE` is deasserted.
- `PSEL` is deasserted if there is no subsequent transfer.
- If another transfer follows to the same peripheral, `PSEL` can remain asserted.

#### Timing

```text
             Setup Phase          Access Phase
                 |                     |
PCLK       _____|‾‾‾‾‾|_____|‾‾‾‾‾|_____
                 T1                    T2

PSEL        ______‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾______

PENABLE     ______________‾‾‾‾‾‾‾‾________

PWRITE      ______‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾______
             WRITE

PADDR       ______<------ Address ------>____

PWDATA      ______<----- Write Data ---->___

PREADY      ______________‾‾‾‾‾‾‾‾________
                         Transfer
                         Complete



dscdsvsdvfdvfv
