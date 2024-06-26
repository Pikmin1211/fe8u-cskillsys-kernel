# ROM space

ROM space distribution is configured in [config-memmap.h](../include/Configs/config-memmap.h)

| Address   | Size    | Usage
| -------   | ------- | -----
| 0x0B2A604 | 0xD5DFC | Kernel
| 0x0EFB2E0 | 0xE4D20 | Fonts
| 0x1000000 | ---     | ***reserved for DEMO***

For kernel, we mainly use free-space starting from `0xB2A604`.

Free space at `0x0EFB2E0` is used to [insert UTF8 characters](../Fonts/Fonts.event) for multi-languge support, but that space is free on default. For more infomation, see [Multi-Language.md](./Multi-Language.md).

In order to co-work on FEBuilderGBA and make DEMO based on kernel, we need also put some inportant data at the fixed location:

### a). Magic pattern

[A serial of characters](../main.event#L11) is set at the head of kernel free-space (`0xB2A604`), which can make it as an identifier for [FEBuilder patch](../Patches/PATCH_SkillInfo.txt#L4).

### b). Pointer list

There is a pointer list after magic pattern, start at `0xB2A614` with size = `0x400`. Both wizardry c-hacks and FEBuilder Patches may find the data via such list, so that you can expand the data via FEB.

### c). Text table

TextTable is repointed at a fixed location behind the pointer list, [0xB2AA14](../include/Configs/config-memmap.h#38) with [size = `0x1300 * sizeof(uintptr_t)`](../Repoint/RepointText/RepointText.event#L9) free space allocated. If you want to make DEMO based on kernel, it is recommanded to directly use the kernel allocated text table rather than re-repoint it by such operations:

```c
#include "fe8-kernel-beta.ref.event" // auto generated by kernel
#define TextTable NewTextTable
#include "Tools/Tool Helpers.txt"    // EventAssembler helper
```


# RAM space

RAM space distribution is configured in [usr-defined.s](../include/Configs/usr-defined.s)

| Address    | Size    | Usage
| -------    | ------  | -----
| 0x02026E30 | 0x2028  | Kernel
| 0x0203F000 | 0x1000  | ***reserved for DEMO***

Since all source codes are all compiled at once, CHAX can offer a better Free-RAM-Space control method.
Free-RAM-Space, unused memory from vanilla is collected by wizardries and now can be refered by [StanH's DOC](https://github.com/StanHash/DOC/blob/master/FREE-RAM-SPACE.md). Here we mainly use space start at `0x02026E30` with size `0x2028`, which is the debug print buffer in vanilla (and unused). They are all defined in [usr-defined.s](../include/Configs/usr-defined.s).

We have also add a detection for RAM space overflow, CHAX may auto detect on overflow error on [game-init](../Wizardry/Common/GameInitHook/source/GameInit.c#L14).

## Example

Here is an example to alloc ram spaces in kernel:

Suppose you want a 4 Byte RAM space (`u8 NewAlloc4Bytes[4]`),

1. Get into [usr-defined.ref.s](../include/Configs/usr-defined.s).
2. Replace `FreeRamSpaceTail` to `NewAlloc4Bytes`.
3. Update `FreeRamSpaceTail` to the end of `NewAlloc4Bytes` (aka `SET_DATA FreeRamSpaceTail, NewAlloc4Bytes + 0x4`).
4. Declare such variable in your own C file, `extern u8 NewAlloc4Bytes[4];`
