---
title: Discord Hard Drive
layout: default
filename: discord_disk
order: 2025
--- 

# Discord Hard Drive
I used [nbdkit](https://libguestfs.org/nbdkit.1.html) and Discord's API to create a virtual "hard drive" which uses a Discord bot to store data in Discord messages, allowing theoretically unlimited (but practically very-limited) storage.

This project is inspired by suckerpinch's video: [Harder Drives](https://www.youtube.com/watch?v=JcJSW7Rprio). 


## Demo
A quick demo of this project is available here:
<iframe width="640" height="385" src="https://www.youtube.com/embed/nOIQ9eKzWVw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Technical Details

### Storage Format
Of course, discord allows file attachments, so I could have completed this project using discord file attachments to store the data. However, as a project primarily intended to be impractical, I decided to store my data in the actual text _messages_ of discord. Unfortunately, Discord has a 2000 character limit on message lengths, so we can't store an arbitrarily large amount of data.

Fortunately for us, with a little bit of testing, it seems Discord counts message lengths in terms of the number of UTF-16 code points needed to represent the text, meaning we could theoretically store a maximum of 4000 bytes per message. However, using this naively would result in invalid unicode, as not every possible 16-bit character actually corresponds to a usable unicode character. Therefore, we use the [braille Unicode Block](https://en.wikipedia.org/wiki/Braille_Patterns), which gives us a very convenient way to represent our data. Each braille character is represented by 2 bytes, with the first being `0x28`, and the second corresponding to a bit-map of which dots are active. Therefore, `0x2800` is an empty braille character, `0x28FF` is a braille character with all 8 dots shown, and so on. Therefore, with this format, every Discord message can hold 2000 bytes of data.

### Data Structures
We would like to abstract this as a block device in Linux. This means, that we essentially need to create the illusion of an arbitrarily large tape of bytes, which the operating system can read from or write to at any offset, and for any length. We could naively implement this as a series of messages, with each message storing 1000 bytes of raw data, but Discord's API doesn't allow us to simply get the nth message in a channel. It _does_ allow us to get the 100 messages after a given message, but this format would either mean storing the message ID of each 100th message, or slowly scanning through all messages to find a particular offset. This would result in either `O(n)` local storage being used, or `O(n)` time to find the offset in Discord for each read or write.

Therefore, we instead group messages in 100 message "blocks", each preceded by a "level 0 header". Therefore, we can get the 100 messages after a certain level 0 header to get 200,000 bytes of data at a time. Instead of storing each 8-byte message ID of the level 0 headers locally, we can store 250 of them in another discord message, a "level 1 header". Therefore, each level 1 header corresponds to 250 * 100 * 2000 = 50,000,000 bytes of data. When we want to store more than that, we can then create "level 2 headers" to store the message IDs of level 1 headers, and so on. This is essentially how pointers and tree data structures would be represented in RAM, and gives us a convenient `O(1)` local storage space (as we only need to store the singular message ID of the topmost header), and `O(log(n))` read/write time. It's worth noting that (much like a real hard drive), reading is much faster than writing, because Discord's API allows us to read 100 messages at a time, but only allows us to write a single message at a time.

### Laziness
We can note that the actual order of the blocks and header messages in Discord is ultimately arbitrary. We find the blocks based on their message IDs, not their locations. Therefore, to avoid needing to actually send several gigabytes of data to Discord when we initialize the disk, we can implement the optimization of only actually creating the headers and message blocks corresponding to a particular section of addresses upon the first write to it. We then only need to edit messages when a write occurs to a sector _which has already been written to previously_. When we need to read from a sector which isn't represented by a message block yet, we simply return all `0`s for the sections not yet represented, without actually creating discord messages for them, as we know that they have never been written to.

## Future Development
Given that the primary intent of this project is to be funny and amusing, I consider it complete, so I don't have plans to return to it in the future. However, this project has given me a bit of an insight into the low-level details of how partitions and file systems look as raw binary data, and how the Linux kernel reads and interprets that data, which I plan to continue learning about and exploring.
