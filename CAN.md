# CAN tutorial

# Introduction
[Video Link](https://www.csselectronics.com/screen/page/simple-intro-to-can-bus/language/en)

# CAN bus in STM32

HAL driver:
```
/**
  ******************************************************************************
  * @file    stm32f4xx_hal_can.c
  * @author  MCD Application Team
  * @brief   CAN HAL module driver.
  *          This file provides firmware functions to manage the following
  *          functionalities of the Controller Area Network (CAN) peripheral:
  *           + Initialization and de-initialization functions
  *           + Configuration functions
  *           + Control functions
  *           + Interrupts management
  *           + Callbacks functions
  *           + Peripheral State and Error functions
  *
  @verbatim
  ==============================================================================
                        ##### How to use this driver #####
  ==============================================================================
    [..]
      (#) Initialize the CAN low level resources by implementing the
          HAL_CAN_MspInit():
         (++) Enable the CAN interface clock using __HAL_RCC_CANx_CLK_ENABLE()
         (++) Configure CAN pins
             (+++) Enable the clock for the CAN GPIOs
             (+++) Configure CAN pins as alternate function open-drain
         (++) In case of using interrupts (e.g. HAL_CAN_ActivateNotification())
             (+++) Configure the CAN interrupt priority using
                   HAL_NVIC_SetPriority()
             (+++) Enable the CAN IRQ handler using HAL_NVIC_EnableIRQ()
             (+++) In CAN IRQ handler, call HAL_CAN_IRQHandler()

      (#) Initialize the CAN peripheral using HAL_CAN_Init() function. This
          function resorts to HAL_CAN_MspInit() for low-level initialization.

      (#) Configure the reception filters using the following configuration
          functions:
            (++) HAL_CAN_ConfigFilter()

      (#) Start the CAN module using HAL_CAN_Start() function. At this level
          the node is active on the bus: it receive messages, and can send
          messages.

      (#) To manage messages transmission, the following Tx control functions
          can be used:
            (++) HAL_CAN_AddTxMessage() to request transmission of a new
                 message.
            (++) HAL_CAN_AbortTxRequest() to abort transmission of a pending
                 message.
            (++) HAL_CAN_GetTxMailboxesFreeLevel() to get the number of free Tx
                 mailboxes.
            (++) HAL_CAN_IsTxMessagePending() to check if a message is pending
                 in a Tx mailbox.
            (++) HAL_CAN_GetTxTimestamp() to get the timestamp of Tx message
                 sent, if time triggered communication mode is enabled.

      (#) When a message is received into the CAN Rx FIFOs, it can be retrieved
          using the HAL_CAN_GetRxMessage() function. The function
          HAL_CAN_GetRxFifoFillLevel() allows to know how many Rx message are
          stored in the Rx Fifo.

      (#) Calling the HAL_CAN_Stop() function stops the CAN module.

      (#) The deinitialization is achieved with HAL_CAN_DeInit() function.


      *** Polling mode operation ***
      ==============================
    [..]
      (#) Reception:
            (++) Monitor reception of message using HAL_CAN_GetRxFifoFillLevel()
                 until at least one message is received.
            (++) Then get the message using HAL_CAN_GetRxMessage().

      (#) Transmission:
            (++) Monitor the Tx mailboxes availability until at least one Tx
                 mailbox is free, using HAL_CAN_GetTxMailboxesFreeLevel().
            (++) Then request transmission of a message using
                 HAL_CAN_AddTxMessage().


      *** Interrupt mode operation ***
      ================================
    [..]
      (#) Notifications are activated using HAL_CAN_ActivateNotification()
          function. Then, the process can be controlled through the
          available user callbacks: HAL_CAN_xxxCallback(), using same APIs
          HAL_CAN_GetRxMessage() and HAL_CAN_AddTxMessage().

      (#) Notifications can be deactivated using
          HAL_CAN_DeactivateNotification() function.

      (#) Special care should be taken for CAN_IT_RX_FIFO0_MSG_PENDING and
          CAN_IT_RX_FIFO1_MSG_PENDING notifications. These notifications trig
          the callbacks HAL_CAN_RxFIFO0MsgPendingCallback() and
          HAL_CAN_RxFIFO1MsgPendingCallback(). User has two possible options
          here.
            (++) Directly get the Rx message in the callback, using
                 HAL_CAN_GetRxMessage().
            (++) Or deactivate the notification in the callback without
                 getting the Rx message. The Rx message can then be got later
                 using HAL_CAN_GetRxMessage(). Once the Rx message have been
                 read, the notification can be activated again.


      *** Sleep mode ***
      ==================
    [..]
      (#) The CAN peripheral can be put in sleep mode (low power), using
          HAL_CAN_RequestSleep(). The sleep mode will be entered as soon as the
          current CAN activity (transmission or reception of a CAN frame) will
          be completed.

      (#) A notification can be activated to be informed when the sleep mode
          will be entered.

      (#) It can be checked if the sleep mode is entered using
          HAL_CAN_IsSleepActive().
          Note that the CAN state (accessible from the API HAL_CAN_GetState())
          is HAL_CAN_STATE_SLEEP_PENDING as soon as the sleep mode request is
          submitted (the sleep mode is not yet entered), and become
          HAL_CAN_STATE_SLEEP_ACTIVE when the sleep mode is effective.

      (#) The wake-up from sleep mode can be trigged by two ways:
            (++) Using HAL_CAN_WakeUp(). When returning from this function,
                 the sleep mode is exited (if return status is HAL_OK).
            (++) When a start of Rx CAN frame is detected by the CAN peripheral,
                 if automatic wake up mode is enabled.

  *** Callback registration ***
  =============================================

  The compilation define  USE_HAL_CAN_REGISTER_CALLBACKS when set to 1
  allows the user to configure dynamically the driver callbacks.
  Use Function @ref HAL_CAN_RegisterCallback() to register an interrupt callback.

  Function @ref HAL_CAN_RegisterCallback() allows to register following callbacks:
    (+) TxMailbox0CompleteCallback   : Tx Mailbox 0 Complete Callback.
    (+) TxMailbox1CompleteCallback   : Tx Mailbox 1 Complete Callback.
    (+) TxMailbox2CompleteCallback   : Tx Mailbox 2 Complete Callback.
    (+) TxMailbox0AbortCallback      : Tx Mailbox 0 Abort Callback.
    (+) TxMailbox1AbortCallback      : Tx Mailbox 1 Abort Callback.
    (+) TxMailbox2AbortCallback      : Tx Mailbox 2 Abort Callback.
    (+) RxFifo0MsgPendingCallback    : Rx Fifo 0 Message Pending Callback.
    (+) RxFifo0FullCallback          : Rx Fifo 0 Full Callback.
    (+) RxFifo1MsgPendingCallback    : Rx Fifo 1 Message Pending Callback.
    (+) RxFifo1FullCallback          : Rx Fifo 1 Full Callback.
    (+) SleepCallback                : Sleep Callback.
    (+) WakeUpFromRxMsgCallback      : Wake Up From Rx Message Callback.
    (+) ErrorCallback                : Error Callback.
    (+) MspInitCallback              : CAN MspInit.
    (+) MspDeInitCallback            : CAN MspDeInit.
  This function takes as parameters the HAL peripheral handle, the Callback ID
  and a pointer to the user callback function.

  Use function @ref HAL_CAN_UnRegisterCallback() to reset a callback to the default
  weak function.
  @ref HAL_CAN_UnRegisterCallback takes as parameters the HAL peripheral handle,
  and the Callback ID.
  This function allows to reset following callbacks:
    (+) TxMailbox0CompleteCallback   : Tx Mailbox 0 Complete Callback.
    (+) TxMailbox1CompleteCallback   : Tx Mailbox 1 Complete Callback.
    (+) TxMailbox2CompleteCallback   : Tx Mailbox 2 Complete Callback.
    (+) TxMailbox0AbortCallback      : Tx Mailbox 0 Abort Callback.
    (+) TxMailbox1AbortCallback      : Tx Mailbox 1 Abort Callback.
    (+) TxMailbox2AbortCallback      : Tx Mailbox 2 Abort Callback.
    (+) RxFifo0MsgPendingCallback    : Rx Fifo 0 Message Pending Callback.
    (+) RxFifo0FullCallback          : Rx Fifo 0 Full Callback.
    (+) RxFifo1MsgPendingCallback    : Rx Fifo 1 Message Pending Callback.
    (+) RxFifo1FullCallback          : Rx Fifo 1 Full Callback.
    (+) SleepCallback                : Sleep Callback.
    (+) WakeUpFromRxMsgCallback      : Wake Up From Rx Message Callback.
    (+) ErrorCallback                : Error Callback.
    (+) MspInitCallback              : CAN MspInit.
    (+) MspDeInitCallback            : CAN MspDeInit.

  By default, after the @ref HAL_CAN_Init() and when the state is HAL_CAN_STATE_RESET,
  all callbacks are set to the corresponding weak functions:
  example @ref HAL_CAN_ErrorCallback().
  Exception done for MspInit and MspDeInit functions that are
  reset to the legacy weak function in the @ref HAL_CAN_Init()/ @ref HAL_CAN_DeInit() only when
  these callbacks are null (not registered beforehand).
  if not, MspInit or MspDeInit are not null, the @ref HAL_CAN_Init()/ @ref HAL_CAN_DeInit()
  keep and use the user MspInit/MspDeInit callbacks (registered beforehand)

  Callbacks can be registered/unregistered in HAL_CAN_STATE_READY state only.
  Exception done MspInit/MspDeInit that can be registered/unregistered
  in HAL_CAN_STATE_READY or HAL_CAN_STATE_RESET state,
  thus registered (user) MspInit/DeInit callbacks can be used during the Init/DeInit.
  In that case first register the MspInit/MspDeInit user callbacks
  using @ref HAL_CAN_RegisterCallback() before calling @ref HAL_CAN_DeInit()
  or @ref HAL_CAN_Init() function.

  When The compilation define USE_HAL_CAN_REGISTER_CALLBACKS is set to 0 or
  not defined, the callback registration feature is not available and all callbacks
  are set to the corresponding weak functions.

  @endverbatim
  ******************************************************************************
  * @attention
  *
  * <h2><center>&copy; Copyright (c) 2016 STMicroelectronics.
  * All rights reserved.</center></h2>
  *
  * This software component is licensed by ST under BSD 3-Clause license,
  * the "License"; You may not use this file except in compliance with the
  * License. You may obtain a copy of the License at:
  *                        opensource.org/licenses/BSD-3-Clause
  *
  ******************************************************************************
  */
```

# Calculate parameter for CAN
http://www.bittiming.can-wiki.info/

# Motor Documentation
[RoboMaster C620 Brushless Driver Documentation](https://rm-static.djicdn.com/tem/17348/RoboMaster%20C620%20Brushless%20DC%20Motor%20Speed%20Controller%20V1.01.pdf)
