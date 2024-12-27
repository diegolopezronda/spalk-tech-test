# Spalk Tech Test
* This script reads an MPEG-TS file from stdin and outputs a list of PIDs in the file.
* It uses tsdump to find invalid packets and tsanalyze to find PIDs.
* An invalid packet is a packet that does not start with the sync byte 0x47.
* If there are no invalid packets, it outputs a list of PIDs.
* If there are invalid packets, it outputs an error message for each invalid packet and exits with a non-zero status.

## Dependencies
* [FFMPEG](https://www.ffmpeg.org/download.html)
* [TSDuck](https://tsduck.io/download/tsduck/)
  
## Usage
```
cat file.ts | ./mpegts-parser.sh
```

## References
* [MPEG Transport Stream in Wikipedia](https://en.wikipedia.org/wiki/MPEG_transport_stream#Elements)

## Rationale
* The idea behind the code is writing as little as possible and in the most reliable way, taking advantage of existing solutions.
* Before I started the challenge, I wondered if the problem had been sorted by someone else already.
* I also wondered if the challenge was related to a standard.
* Lucky for me, MPEG Transport Stream is a standard, so I read about the format.
* Then, I proceeded to find a well-known library that would help me fulfil the challenge's requirements.
* I found the FFMPEG suite that contains the command `ffprobe`, which does most of the work.
* I didn't find a way to display the expected output using `ffprobe` only; some streams were missing in the success output.
* I used VLC to play the videos, and I could see the streams (both audio and video) that `ffprobe` was prompting.
* I used `hexdump` to display the streams in a human-readable way so I could see the missing steps more clearly, and I found the missing streams.
* I also used `diff` to compare both streams (using the output from `hexdump`), and I discovered that only one byte was different between files.
* That byte was supposed to be a sync byte, but in the failure example, it was tampered with and thus triggered an error when tracing. 
* After those discoveries, I wondered if there was a more refined tool to solve the problem. FFMPEG has been there for a while, so it should be a fork somewhere.
* I found TSDuck, which is another suite similar to FFMPEG.
* When using TSDuck, `tsanalyze` was useful for finding all the PIDs of the success output.
* `tsdump` helped find the frame mistakes.
* The remaining part of the solution was refining the output using unix commands.

## Challenge
### Preamble
* The purpose of this test is for us to get a sense of your current code style 
and provide talking points for discussion during the interview process.
* We expect you to spend a maximum of 4 hrs completing this test, please do not feel compelled to pay more if you have not completed the task.
* Ideally you complete the task but if you cannot please deliver your attempt 
as completely as possible.
* Please complete the test using any programming language you are comfortable 
with.
* Please provide instructions to compile and run the solution alongside any 
code you produce.
### Introduction
You may be familiar with the MPEG Transport Stream format, it is a streaming 
standard used by broadcasters to distribute broadcast content from one point to 
another. Some useful aspects of the standard are:
1. The streaming format is made up of individual packets
2. Each packet is 188 bytes long
3. Every packet begins with a "sync byte" which has hex value 0x47. Note this 
is also a valid value in the payload of the packet.
4. Each packet has an ID, known as the PID that is 13 bits long. The PID is 
stored in the last 5 bits of the second byte, and all 8 bits of the third byte 
of a packet eg:

| **BYTE**  | 0 |   |   |   |   |   |   |   | 1 |   |   |   |   |   |   |   | 2 |   |   |   |   |   |   |   | 3 |   |   |   | ... | 187 |   |   |   |
|-----------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|-----|-----|---|---|---|
| **BIT**   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 0 | 1 | 2 | 3 | ... | 4   | 5 | 6 | 7 |
| **VALUE** | 0 | 1 | 0 | 0 | 0 | 1 | 1 | 1 |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | ... |     |   |   |   |
| **FIELD** | A | A | A | A | A | A | A | A | B | C | D | E | E | E | E | E | E | E | E | E | E | E | E | E | F | F | F | F | ... | F   | F | F | F |

Fields:
* A: Sync Byte
* B: Transport error indicator (TEI)
* C: Payload unit start indicator (PUSI)
* D: Transport priority
* E: Payload

Spalk is exploring the possibility of launching a feature that accepts uploaded 
and streamed MPEG Transport Stream files from our users. As a part of this we 
need to validate that the uploaded files are valid.

### Task
Implement a parser that 
* reads a byte stream from standard input:

```
$ cat some
test
file.ts | ./mpegts-parser
```
* Parses the byte stream to ensure it conforms to the criteria above:
  * The byte stream will contain a series of 188 byte packets, and each packet 
    begins with the sync byte (possibly excepting the first packet)
  * Parse and report the Packet ID (PID) of each packet
  * Will successfully parse a valid stream that has a partial first packet 
    (less than 188 bytes). If the first packet is not complete, it should be 
    discarded.
  * Exits with a success code (0) if the byte stream conforms to the criteria, 
    and a failure code (1) if it does not.
* Outputs a summary:
  * If there are any errors, exit and print the index and byte offset of the 
    first TS packet where the error occurs.
```
$ cat some
test
file.ts | ./mpegts-parser
Error: No sync byte present in packet 1203, offset 226164
$ echo $?
1
```
  * If stdin is closed and there are no errors, lists all the unique PIDs 
    present in the stream in ascending order in hex. Eg:
```
$ cat some
test
file.ts | ./mpegts-parser
0x1010
0x1020
0x1030
0x1040
$ echo $?
0
```

There are example TS files and expected output available in the tech test repo 
here:
[https://github.com/SpalkLtd/tech-test](https://github.com/SpalkLtd/tech-test)

### Deliverables
* Code for execution
* Any supporting documentation required

### What happens next?
* If you have any questions please feel free to contact [michael@spalk.co](michael@spalk.co) 
  with them.
* We will run your code to test for:
  * Compilation/Execution
  * Correctness
  * A part of the interview process will be a code review with members of our 
		team to discuss the code structure and your design choices.
