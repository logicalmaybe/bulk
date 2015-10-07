# How to replace Notepad with Notepad++ globaly.

http://www.howtogeek.com/howto/12617/how-to-replace-notepad-in-windows-7/

In Windows 7, Notepad resides in:
* C:\Windows
* C:\Windows\System32
* C:\Windows\SysWOW64 in 64-bit versions only

----

* Step 1: Permissions => Security (Tab) => Advanced (button at bottom) => Owner (tab) => Edit => Administrator (instead of TrustedInstaller)
* Step 2: OK => Close all Property-Dialogs
* Step 3: Permissions => Security (Tab) => Edit (mid-button) => check "Full control" for Group "Admin"
* Step 4: Replace "notepad.exe" (ms) with "notepad.exe" (++)!
