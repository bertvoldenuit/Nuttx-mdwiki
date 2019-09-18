# Standardize the board initialization

According to Greg:

  > I am trying to standardize to using only stm32_bringup.c with either
  > board_late_initialize() calling stm32_bringup().  But initialization
  > logic in board_app_initialize() only supports one way to initialized. 
  > Putting all initialization logic to stm32_bringup() is more flexible and
  > supports alternative board initialize strategies.  Whenever I work with
  > a board that does initialization within board_app_initialize(), I change
  > it.  I move the initialization to <arch_name>_bringup.c and call `<arch_name>`_bringup().
  > Someday all boards will be converted to use `<arch_name>`_bringup()!  

So let's standardize the board initialization:

| Replace          | by                                                                            |
|------------------|-------------------------------------------------------------------------------|
| `<board_name>` | your board's name   (e.g. `<board_name>`/src --> **nucleo-f4x1re**/src)     |
| `<arch_name>`  | MCU familly or arch   (e.g. `<arch_name>`_bringup.c -> **stm32**_bringup.c) |

First let's check if `<arch_name>`_bringup.c exists, from your home directory type:

```
cd nuttxspace/nuttx/configs/<board_name>/src
ls <arch_name>_bringup.c
```

If it does not exist then copy it:

```
cd nuttxspace/nuttx/configs/<board_name>/src
cp ../../stm32f103-minimum/src/stm32_bringup.c <arch_name>_bringup.c
```

then edit `<arch_name>`_bringup.c, and make the following change:

`#include "stm32f103_minimum.h"`

by

`#include "<board_name>.h"`


Edit `<arch_name>`_boot.c and change the following:

```
void board_initialize(void)
{
  /* Perform NSH initialization here instead of from the NSH.  This
   * alternative NSH initialization is necessary when NSH is ran in user-space
   * but the initialization function must run in kernel space.
   */

#if defined(CONFIG_NSH_LIBRARY) && !defined(CONFIG_LIB_BOARDCTL)
  board_app_initialize(0);
#endif
}
```

Change the above by the following. If the above does not exist just add the following at the end:

```
#ifdef CONFIG_BOARD_INITIALIZE
void board_initialize(void)
{
  /* Perform NSH initialization here instead of from the NSH.  This
   * alternative NSH initialization is necessary when NSH is ran in user-space
   * but the initialization function must run in kernel space.
   */

#if defined(CONFIG_NSH_LIBRARY) && !defined(CONFIG_LIB_BOARDCTL)
  board_app_initialize(0);
#endif
  
#ifndef CONFIG_LIB_BOARDCTL
  /* Perform board initialization here instead of from the board_app_initialize(). */

  (void)<arch_name>_bringup();
#endif
}
```

Edit `<board_name>`.h and add the following below Public Functions:

```
/****************************************************************************
 * Name: <arch_name>_bringup
 *
 * Description:
 *   Perform architecture specific initialization
 *
 *   CONFIG_LIB_BOARDCTL=y:
 *     If CONFIG_NSH_ARCHINITIALIZE=y:
 *       Called from the NSH library (or other application)
 *     Otherwise, assumed to be called from some other application.
 *
 *   Otherwise CONFIG_BOARD_INITIALIZE=y:
 *     Called from board_initialize().
 *
 *   Otherise, bad news:  Never called
 *
 ****************************************************************************/

int <arch_name>_bringup(void);
```