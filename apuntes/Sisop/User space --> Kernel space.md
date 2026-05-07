#sisop 

---

- **The Trigger:** A user space process executes the `syscall` instruction.
- **The Mode Switch (Hardware):** The CPU catches the trap, immediately halts the user program, and flips its internal privilege level from Ring 3 to Ring 0.
- **The Context Save (Software):** The kernel wakes up and saves the CPU's current state (the user program's registers) to the kernel stack so nothing gets corrupted.
- **The Execution (Software):** The kernel performs the actual privileged task (like writing to the disk).
- **The Context Restore (Software):** The kernel pulls the saved state off the stack and carefully puts it back into the CPU registers.
- **The Return (Hardware/Software):** The kernel executes a return instruction. The CPU flips the mode back to Ring 3, handing control back to the user space process exactly where it left off.
