# Operations
P3 handles most changes to the world and its elements through *operations*.
An operation is a 20 bytes wide struct, where the first u32 denotes the operation's type, and the following 16 bytes contain the operation's arguments.

## Scheduling
While the game executes operation handlers directly sometimes, usually operations are scheduled by inserting them into the *pending operations* queue in the static `operations` struct at `0x006DF2F0`:
```
00000000 operations      struc ; (sizeof=0x948, mappedto_126)
[...]
00000048 field_48_pending_operations operation_switch_input_container ?
0000046C field_46C       dd ?
00000470 field_470_pending_operations_in_use dd ?
00000474 field_474_current_operations operation_switch_input_container ?
[...]
```
The `operation_switch_input_container` struct contains an array of 53 operations.
The function `schedule_operation` at `0x00543F10` inserts an operation at the next free position, creating a new `operation_switch_input_container` and appending it to the last full one if necessary.

## Execution
The function `execute_operations` at `0x00546870` removes up to 53 operations from `operations` and executes them.

## Debugging
The following IDC script adds scripted breakpoints to the executing and scheduling functions, allowing the investigation of P3's operation behavior:
```
static handle_operation_switch() {
    auto ptr = GetRegValue("esi");
    auto operationSwitchInput = OperationSwitchInput(ptr);
    // Ignore noisy operations
    if (operationSwitchInput.opcode() == 0x94) {
        return 0;
    }
    if (operationSwitchInput.opcode() == 0x24) {
        return 0;
    }
    if (operationSwitchInput.opcode() == 0x7B) {
        return 0;
    }
    
    Message(operationSwitchInput.toString());

    return 0;
}

static handle_insert_into_pending_operations_wrapper() {
    auto ptr = GetRegValue("eax");
    auto operationSwitchInput = OperationSwitchInput(ptr);
    // Ignore noisy operations
    if (operationSwitchInput.opcode() == 0x94) {
        return 0; // weird thing
    }

    Message("handle_insert_into_pending_operations_wrapper %s", operationSwitchInput.toString());

    return 0;
}

static main() {
    auto bp = AddBpt(0x0053576B);
    SetBptCnd(0x0053576B, "handle_operation_switch()");
    auto bp2 = AddBpt(0x0054AABD);
    SetBptCnd(0x0054AABD, "handle_insert_into_pending_operations_wrapper()");
}
```
## Identified Operations
The following operations have been identified:

|Opcode|Task|
|-|-|
|0x00|Move Ship|
|0x01|Sell Wares from Ship|
|0x02|Buy Wares to Ship|
|0x03|Repair Ship|
|0x04|Hire Sailors|
|0x06|Dismiss Captain|
|0x15|Create Convoy|
|0x16|Disband Convoy|
|0x1b|Move Wares|
|0x1d|Repair Convoy|
|0x24|Build Town Wall|
|0x25|Build Road|
|0x26|Demolish Building|
|0x29|Grant Loan|
|0x2c|Build Ship|
|0x30|Feed the Poor|
|0x31|Donate to Church Extension|
|0x32|Donate to Church|
|0x37|Join Guild|
|0x39|Bathe|
|0x41|Form Militia Squad|
|0x42|Bath House Bribe Success|
|0x43|Bath House Bribe Failure|
|0x48|Make Town Hall Offer|
|0x52|Tavern Interaction|
|0xc2|Autosave|
|0xc4|Advance Time|
|0x9f|Start Ship Combat|
|0x96|Steer Manually|
