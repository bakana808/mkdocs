# Buffered Input System

Many games have a buffered input system, such as fighting games. This reduces the amount of dropped inputs by allowing a "buffer" (a window of time, usually in frames) where if the player pressed the input too early but was within the buffer, the input will still register as pressed.

For example, consider a platform game where the player can only jump from the floor. The player is falling from the air, and tried to press jump as they were about to land.

With a normal input system, if the player pressed jump at any time before the time they actually landed, even if it was a couple frames early, the input would be dropped.

With a buffered input system, we can set a buffer for which the input would still go through even if it was early. For example, with a 4-frame buffer, the player could press jump 4 frames before they landed and the jump will still happen.