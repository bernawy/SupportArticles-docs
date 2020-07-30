---
title: IME may crash when processing a window message sent from another thread
description: Describes the scenario where an Input Method Editor (IME) may crash when processing a window message sent from another thread, where the window procedure handling the message calls an Imm* function, such as ImmSetOpenStatus.
ms.date: 
ms.prod-support-area-path: 
ms.reviewer: koichm, davean
---
# IME crashes when processing a window message sent from another thread

Describes a scenario where an Input Method Editor (IME) crashes while processing a window message sent from another thread, and where the window procedure handling the message calls an `Imm*` function such as `ImmSetOpenStatus`.

## Symptoms

An application using an IME crashes under one of these conditions:

- The window procedure associated with an IME window processes a window message.
- A thread sends a window message to a window owned by the thread that owns the IME window.
- The window procedure handling the window message sent by the other thread calls an `Imm*` function, such as `ImmSetOpenStatus`.

## Cause

Some IMEs included with Windows 10 may call the `PeekMessage` function while processing another window message. `PeekMessage` will process any pending window messages sent by other threads. This can result in a reentrancy issue when the window procedure processing the sent message calls an `Imm*` function and leaves the IME in an unexpected state.

## Workaround

Avoid calling `Imm*` functions while the IME is processing another window message.

## More information

The [`PeekMessage`](https://docs.microsoft.com/windows/win32/api/winuser/nf-winuser-peekmessagea) documentation contains the following:

During this call, the system delivers pending and non-queued messages sent to windows owned by the calling thread using the [`SendMessage`](https://docs.microsoft.com/windows/desktop/api/winuser/nf-winuser-sendmessage), [`SendMessageCallback`](https://docs.microsoft.com/windows/desktop/api/winuser/nf-winuser-sendmessagecallbacka), [`SendMessageTimeout`](https://docs.microsoft.com/windows/desktop/api/winuser/nf-winuser-sendmessagetimeouta), or [`SendNotifyMessage`](https://docs.microsoft.com/windows/desktop/api/winuser/nf-winuser-sendnotifymessagea) function. The first queued message matching the specified filter is retrieved. The system may also process internal events. If no filter is specified, messages are processed in this order:

- Sent messages
- Posted messages
- Input (hardware) messages and system internal events
- Sent messages (again)
- [`WM_PAINT`](https://docs.microsoft.com/windows/desktop/gdi/wm-paint) messages
- [`WM_TIMER`](https://docs.microsoft.com/windows/desktop/winmsg/wm-timer) messages

> [!NOTE]
> Sent window messages are non-queued messages. The message filter specified when calling PeekMessage call doesn't apply to sent messages.