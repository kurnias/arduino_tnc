Arduino TNC
===========
What I fixed first:

ISR/shared-state safety: added volatile to the key ISR-shared variables and ring-buffer pointers.
UART RX ring buffer: added proper full-buffer detection so unread bytes are not silently overwritten.
RX frame buffer safety: added bounds checking so msg[] cannot overflow on malformed/long frames.
Removed sei() from inside the sample ISR.
Safer port handling: replaced raw PORTB = ... / PORTB &= 0x00 style DAC writes with a masked helper so the code stops clobbering unrelated bits.
Added missing prototypes and #include <string.h>.
Replaced legacy SIGNAL(...) handlers with ISR(...).
KISS parser: rewrote it into a more explicit state machine for FEND / FESC handling.
Added RX FCS validation before forwarding a received AX.25 frame to the host.

Important caveat:

I kept the original demodulator / timing architecture intact on purpose, so this is a high-risk-bug-fix pass, not a full redesign.
I did not hardware-test or compile-test it against your exact Arduino toolchain here, so treat it as a strong corrected draft, not yet field-validated firmware.

The one hardware assumption I changed deliberately:

On the ATmega328P-style build, your original code used PORTB both for the DAC ladder and for DCD/HB signaling, which is fundamentally unsafe. In this cleaned version I made DCD/HB no-ops on that build unless you remap them to another port.

Next best step is a second pass where I:

normalize naming (ARD_DUE is misleading for ATmega328P),
split RX/TX and KISS logic into cleaner modules,
and make the timer setup much more explicit and auditable.
