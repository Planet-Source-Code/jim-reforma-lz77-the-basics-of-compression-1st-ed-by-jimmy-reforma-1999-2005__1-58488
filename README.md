<div align="center">

## LZ77 the basics of compression \(1st ed\.\)  by Jimmy Reforma 1999\-2005


</div>

### Description

Introduction

In this article I'll present the basics of lossless compression, also called text compression. This scheme, lz77, is very used because it's easy to implement and also it's fast. (if you improve it, of course)

This is the second version of this article, if you've read the second version, you'll notice that is new version is bigger, in fact from 15k to 33k, more than twice, and its better than the first one. Also I recommend reading it, even if you've read the first version, because you'll learn even more. Even this new version is in html format. This is a second version corrected. (have a look at the date at the end)! You can download complete article of this lossless compression at http://www.dakila7forums.ne1.net/
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Jim Reforma](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/jim-reforma.md)
**Level**          |Advanced
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |VB 3\.0, VB 4\.0 \(16\-bit\), VB 4\.0 \(32\-bit\), VB 5\.0, VB 6\.0, ASP \(Active Server Pages\) , VBA MS Access, VBA MS Excel
**Category**       |[Miscellaneous](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/miscellaneous__1-1.md)
**World**          |[Visual Basic](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/visual-basic.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/jim-reforma-lz77-the-basics-of-compression-1st-ed-by-jimmy-reforma-1999-2005__1-58488/archive/master.zip)





### Source Code

```
Introduction
In this article I'll present the basics of lossless compression, also called text compression. This scheme, lz77, is very used because it's easy to implement and also it's fast. (if you improve it, of course)
This is the second version of this article, if you've read the first version, you'll notice that is new version is bigger, in fact from 15k to 33k, more than twice, and its better than the first one. Also I recommend reading it, even if you've read the first version, because you'll learn even more. Even this new version is in html format. This is a new version corrected. (have a look at the date at the end)
The way I present you Lz77, will not lead you to do an archiver, but may be very interesting for internal data of your programs. It will have a slow compression, a fast decompression and a good ratio. (till N comparisons per byte, where N is the size of the sliding window) Its decompression is the fastest, unless lzrw which perhaps is faster. So it's the perfect algorithm for internal data of program. Also it's the better algorithm for little amounts of data (Haven't you see the 4k intro Mesha by Picard? ;-) And also it's free of patents. If you already have implemented lz77 read the section How to improve it probably you'll find something interesting.
Now enjoy the article, and let me know all the errors that it could have.
Theory
In 1977 Abraham Lempel and Jacob Ziv presented their dictionary based scheme for text compression. (in fact text compression refers to lossless compression for all possible data) Till the date all the compression algorithms developed were mainly statical compressors. The new scheme was called lz77. (for obvious reasons) It always outputted offset and lengths to the previous text seen. Also it outputted the next byte after the match, because the context (last bytes seen) of this byte is the phrase, and if it wasn't part of the match (the phrase), then it will not probably compressed, so, why wasting time trying to find a match for it? (and space also)
In 1982 James Storer and Thomas Szymanski basing on the work of Lempel and Ziv, presented their scheme, Lzss. The main difference is in the output, lz77 always outputted an offset/length pair, even if the match was only one byte (in this case we were using more than 8 bits to represent a byte) so Lzss uses another trick to improve it, it uses bit flags, they are just one bit that tells what the next data is, a literal (a byte), or a pair of offset/length. And that's what we actually use, but lzss it's commonly called lz77, so we'll call it lz77 from this point at on, but remember that it can also be named Lzss. Lzss also can use binary search trees or suffix trees, for doing a faster search. (which is the bottleneck of lz77)
What's the theory? It's very simple and intuitive. When you find a match (aka phrase, a group of bytes which have been already see in the input file) instead of writing those bytes you write the offset and the length of the repetition: where is it and how long is it.
This is a dictionary based scheme, because you keep a dictionary (the sliding window) and you make references to it. (with the offset/length pair) This version, lz77, uses a sliding window, which has a maximum length, so this window can't be the whole file, instead the sliding window holds the last bytes 'seen'.
Lz77
Imagine you are compressing a text: "ab ab" you read till "ab " and write it uncompressed, then you read "ab" and then you write the following: in offset 0 there are two repeated bytes. And how the decompression works? easy, you first read "ab " and then the offset and length, and you copy the bytes from there, look:
Get 'a'. "a"
Get 'b'. "ab"
Get ' '. "ab "
Get Offset and length. Copy two bytes from position 0. ("ab") "ab ab"
But how the decompressor can now if there is an offset/length or an uncompressed byte? simply, we use a prefix, a prefix is a bit that is like a switch with two cases and let us know how the following data is. If its a 0 then it's and uncompressed byte, if it's 1 then it's an offset/length pair. Those prefixes are also called flags.
This pair, offset and length, is called code word. A code word is a group of bits (or bytes) that contains some kind of information used by both the compressor and decompressor. (aka codec) The other possible output of lz77 is a literal. A literal is just a byte uncompressed. So the output of lz77 is of three kinds:
Literals. They are just uncompressed bytes.
Code words. In our case they are pairs of offset and length.
Flags. They tell us if the following data is a literal or a codeword.
And now as an example let's compress again our string and do the 'real' output of lz77:
Get 'a'. No match. Flag 0. Literal 'a'.
Get 'b'. No match. Flag 0. Literal 'b'.
Get ' '. No match. Flag 0. Literal ' '.
Get 'a'. Match. Flag 1. Code word: offset = 0 length = 2
As you see the flags only may have 2 states, 1 or 0. So we only need 1 bit for representing them. Now we can't (we shouldn't) output the flag as a whole byte, we have to work with bits. The output of this compression is called a bit stream, because is a stream of variable length symbols, and the minimum unit is the bit.
Sliding window
If you see the example again, you may ask: where do we have to look for matches? we look backwards, to the data that we've already processed. This is called the sliding window. The sliding window is a buffer which holds the bytes which are before the current position in the file. Every byte we output uncompressed (a literal) is added to the sliding window, and also all the bytes that form a match.
Let's see our example again:
Get 'a'. Sw: "..." No match. Flag 0. Literal 'a'.
Get 'b'. Sw: "a" No match. Flag 0. Literal 'b'.
Get ' '. Sw: "ab" No match. Flag 0. Literal ' '.
Get 'a'. Sw: "ab " Match. Flag 1. Code word: offset=0 length=2
As you can see, when looking for matches we compare the data that we have in our sliding window (Sw) with the data (bytes) at the current position.
So we have to keep one buffer with the data at the current position and another buffer with the sliding window? in some implementations this may be true, but in the implementation that I'll show you, this isn't the way the things are done. Because both the sliding window and the bytes at the current position are nothing else than the file itself, we'll have just one buffer, it'll contain the whole data. Then we just have to care about the pointer to the current position, and the sliding window is just before this pointer. In fact I recommend having the whole file (or at least a big block) and compress it, so you don't have to care about reading more bytes, nor such things.
And now let's talk about the sliding window, how big is it? in fact we can work with the whole file, but think about the offset needed to specify the position of the match. This offset isn't from the position 0 (start of the file) to the match, it's from the current position backwards. So in our example the offset is 3 instead of 0. (thus, when decompressing, the decompressor gets a 3, and subtracts this value to the current offset and it has maked the offset to the match.) As you can see, the bigger the sliding window is, the more bits that we need for saving the pointer, so we have to choose a length for our sliding window. 4096k is widely used, but it's know that the bigger the sliding window is, the better the compression is. So you'll have to choose any length. Let's say we choose length 8192 then we need 13 bits for the offset.
Lengths
Another thing that we must choose is the length of the length. E-) So how many bits will be used for the length? You can choose any length you want. Tuning both the bits for the length and offset you can improve compression in some files and hurt in other files, so if you are designing a compressor just for one file (like in Mesha) you should try the most appropriate values. But now let's use a length from 0-32 so just 5 bits.
Another important thing is the minimum length of a match. In our case we've choosed to spend 13 bits in the offset and 5 in the length, 18 bits, so a match should be at least of 3 bytes. Because if we encode a match of two bytes and spend 18 bytes for both the offset and length we are using 2 bits more. Yes I know 2 bits may seem a very little value, but sometimes you wish your offset could take 2 bits less. ;-)
But now another question arises, if we'll never have matches of 0,1,2 bytes, then why we have space for them in the length?
Let's take profit of every bit. Our length will still have a length of 5 bits, but its range instead of 0-32 will be 3-35.
How shall we do that? easy we just subtract to the length (before saving it) 3, and the decompressor just have to read it
and add 3.
End Marker
Now you should be able to know how the decompression it's done. Note that the decompressor should know how to stop. This may be done in two ways:
You have a symbol that marks the end of the data.
Save along with the bit stream the length of the input file.
I prefer the second method, it's a little bit slower, but at the same time you use it for knowing the end of the data, you may also use it for a possible interface, also it can let you avoid some problems. However, if you want to use a end marker you could use length 0 for it. The way you do it is the following: the range will be from 3-34, in this case we should subtract to it (when saving) the value 2. So the range 1-32 becomes 3-34, and the compressor just have to care about this while compressing, once compression its over, the output the offset/length (you can manage to not put the offset, by putting the length first) and for the length it outputs a 0 value. The only thing which the decompressor should do is every time it reads a length check if it's 0, if it isn't then add to it 2, otherwise, quit decompressing.
Working with bits
As you could see the offsets and lengths are of variable size, and the flags just take 1 bit, so we have to use bits instead of bytes. This is very important in most of the compression algorithms, once you've learn that you don't have to do it again, just like when you learn how to do file Io, you learn it once and use it a lot of times. So let's start with the bit stuff.
If almost all the operations work with bytes and when you save data to a file the minimum unit are bytes how do I use the bits? with a clever use of some instructions.
For this topic I will use ASM, however it also can be done in C. If you don't know ASM, learn it! if you don't want to learn it, then read the articles: 'Lzss' and 'lzp' from the mag Hugi #12 (link at my hpage) where you can find some C code.
Well, let's continue with the operations with bits in ASM. The main idea is to keep a byte and a counter with the bits written, then when you have write 8 bits, you write that byte and start again with another byte. I will use some instructions, be sure to read the explanation of them in the section 'some ASM instructions' Here is the main idea of the put_bits, don't copy it, rewrite it and understand it!
@@put_bits_loop:
push cx
mov bh,_byte_out
mov bl,_byte_in
mov al,bl
shr al,1
xor ah,ah
adc ah,0
mov bl,al
mov cl,_total_bits
shl ah,cl
or bh,al
mov _byte_out,bh
inc _total_bits
cmp _total_bits,8
jne @@no_write_byte
mov di,ptr_buffer
mov es:[di],bh
inc di
mov ptr_buffer,di
inc bytes_writed
mov _byte_out,0
@@no_write_byte:
pop cx
dec cx
jnz @@put_bits_loop 
;the number of bits to write
;the output byte (where to write)
;the input byte (the bits to write)
;we store the byte to read from in al
;we shift to the right al, first bit in the carry flag
;put ah=0
;we add to ah 0 and the carry
;save the input byte
;the bits that we have writed
;put the bit in his position by shifting it to the left
;put the bit in the output byte
;save it
;the bits written
;Do we have write the whole byte?
;nop E-)
;the pointer to the buffer
;save the byte (es is the segment of the buffer)
;next byte in the buffer
;save it for the next time
;when the buffer its full write it to
;a file or something like that so the next time is clear
;we saved it
;more bits to write?
;yes, repeat everything
Well, it's done, as I mentioned I don't like spreading source code, but I thought this was the better way for understanding it. As you see I showed you my putbits routine, I've also done an article about how to improve it, read it, and try to improve it too. The names of the variables are self-explanatory, but anyway:
Variable
Explanation
_byte_out
The byte that will be writed to the output buffer, it holds the bits that we are currently writing.
_byte_in
The byte wich holds the bits that we want to write.
total_bits
The number of bits currently writed, at start 0.
ptr_buffer
 If you are under real mode then it hold the offset to the buffer and es the segment, if you are in pmode it will hold the whole offset. 
When you enter in this routine cx must have the number of bits to write, and _byte_in the bits to write. Be careful, after entering the loop test if cx is 0 because if it's and you don't test you will write 1 bit, then decrement cx, so it's 255, and then you'll write 255 bits! so remember:
test cx,cx
jz @@put_bits_end 
;I will NOT explain that ;-) 
This is the 'structure' (how the bits are written) for a byte:
Bit8
Bit7
Bit6
Bit5
Bit4
Bit3
Bit2
Bit1
When you have write all the bits (ex.: the compression is over) then you have to test if there is some bits waiting for been write, so if there are any (total_bits!=0) then you write the _byte_out, and increment all the pointers so you don't leave any data without writing. Note that this function will fail if you pass it more than 8 bits because it takes the input bits in a byte, not in a word. So if you want to write more than 8 bits then first write the low-part (8 bits) and then the rest, to write and to read it you should use 'ands' 'shl' and 'shr'. Now yoy need the get_bits function... hey! I've explained the basic operations with bits, you should now be able to do that routine yourself, happy coding! E-)
Some ASM instructions
I'll remind you some instructions so you have no problem when doing the putbits and getbits, if you already know that skip to the next section.
Instrucion
Explanation 
Shr
Shift (move) the bits to the right:
 shr al,1 ;first al=11100010b then al=01110001b (cf=0)
 shr al,2 ;first al=01011010b then al=00010111b (cf=1)
 The last bit shifted will go to the carry flag.
 Instead of a direct value you may use cl, and only cl.
 mov cl,2
 shr al,cl ;this will shift al by 2
 Note that the 'new' bit is filled with 0. 
Shl 
The same but to the left. (the carry is changed too)
 shl al,2 ;first al=01001011b then al=00101100b (cf=1) 
Adc
Add with carry. This adds to any register any value and the content
 of the carry flag too.
 adc ah,0 ;first al=0 (cf=1) then al=1
 adc ah,0 ;first al=0 (cf=0) then al=0
 adc ah,3 ;first al=1 (cf=1) then al=5 
Or
This performs a bit operation.
 There is the table: - 0+0=0 - 0+1=1 - 1+1=1 -
 or al,00001111b ;first al=0, then al= 00001111b
 or al,00001111b ;first al=10110010, then al= 10111111b
Maybe you knew how these instructions worked, if you didn't know then learn it.
Output file
Now we should define the output file format, it will be simple, just to fit our needs, the compressed data will be like that: First a word or dword with the size of the original file. (Also if you want, you can put some numbers as Identification, something like "LZ") Then the bit stream, this is the compressed data, it's the bytes containing all the bits that you can read with a getbits. The data in the bitstream is, first a flag bit, which identifies the next data.
Flag 
Next Data
0
Literal 
1
CodeWord 
Remember that the Code Word is the pair offset and length. And the size of every element:
Bits 
Element 
8
Literal (just a byte)
5
Length 
13
Offset
Because this article just pretend to teach the basics, it will not care about things like the Crc-32 or the size of the bit stream,
etc. wich are needed in an archiver.
Pseudo-code
So go now and first of all get the basics of you program, getting the parameters, the work with the files, and do some test with the bits operations, save it to a file, read them, and test if the bits returned are the right ones.
Ok, now I will assume that you have already did all this work. Let's remind how lz77 works, you are in a given position and you try to find backwards (backwards because you are sure that the decompressor already have decoded those bytes when you are in this position) a match, bytes which are equal to the bytes at the current position; if you find them you output a codeword, else you have to output a literal so you can continue compressing.
Well, here it is how we do it:
Save the length of the file to compress
Loop till there is no more bytes to compress
Scan the input buffer starting in current_position-sliding_window_length till the current byte that we are comparing. (Note that the decompressor can't copy bytes from a position where their bytes aren't already defined.)
Have we found a byte equal to the current?
Yes.
Then compare the next byte from the current position with the byte in the next position when we've founded a byte equal to the first.
Continue comparing till you find a byte that isn't equal, but remember keeping the number of bytes which are equal.
Now you have found a byte that isn't equal. Is the number of bytes found more than 3?
Yes. Write the offset of the FIRST byte found, and the number of bytes repeated. (length) Then advance the pointer to the current position with the number of bytes repeated (because we have 'saved' it) and continue searching. (also a 1 flag)
No. continue searching.
No. If you don't find any match then you simply write and uncompressed byte. (also you write a literal if there's no data at the sliding window) (remember to put first the 0 flag)
If you don't exactly know how comparisons are done look at the section Looking for matches. That is all the work. Remember of writing the flags too. Go and implement it, test it with an easy text, for example check it with those strings:
"11 222 11 222" "111222111312221"
Your compressor seem to not have bugs? yes? well, now we have to do a decompressor:
you read it the length of the uncompressed file
Then you loop till you've uncompressed the whole file
Read a bit (the flag)
It's 0
Read 8 bits and write them to the output buffer (remember they are an uncompressed byte) Increment the pointer to the output.
It's 1
Read the whole offset, 13 bits. then the length, copy 'length' byte from 'offset' to the current position, and add to the pointer to the output the 'length'.
Now the compressor and the decompressor are done, well, if you have any bug, and you can't find it look at the section Possible bugs, else go directly to the next section. If you have did the compressor and the decompressor without any bugs, then you have did a good work, congratulations, but there is still a lot of things to do. E-) Hey!, keep up the good work. ;-)
Looking for matches
The way you search the matches is the following, you keep a pointer to the current position. At the start of any itineration, you compute the offset to the Sliding Window. You can easily do this getting the pointer to the current position and subtracting to it the length of the sliding window, in case it underflows (it goes beyond 0) just set it to 0.
Let's say we have a sliding window of 4 bytes long. (So we spend 2 bits to specify this offset, but never do that, this is too little) And we have the following string: "1234567"
Cp: 0. Swp=0-4=0. Current: "1234567" Sliding Window: "..."
Cp: 1. Swp=1-4=0. Current: "234567" Sliding Window: "1"
Cp: 2. Swp=2-4=0. Current: "34567" Sliding Window: "12"
Cp: 3. Swp=3-4=0. Current: "4567" Sliding Window: "123"
Cp: 4. Swp=4-4=0. Current: "567" Sliding Window: "1234"
Cp: 5. Swp=5-4=1. Current: "67" Sliding Window: "2345"
Cp: 6. Swp=6-4=2. Current: "7" Sliding Window: "3456"
Where Cp is the pointer to the current bytes, and Swp the pointer to the start of the sliding window. When using pointers to the whole input file you have to care about the length of the sliding window. You can keep a variable with the length. But I do something different. Let's say we have in Esi the pointer to the start of the sliding window, and in Edi the pointer to the current position, then I compare and look for matches with Esi till it's equal to edi, then it means that we are in the current position, and because this will not be available to the decompressor, then we shouldn't look for matches there. The routine that search matches in the sliding window is called parser. The way I look for matches is the following, I get the byte at the current position, then I search thru the sliding window till I find a byte equal, something like that:
_byte_=*current_pointer;
 for(i=0;i<=sliding_window_length;++i)
 {
 if(_byte_==*sliding_window_pointer)
 {
 //count how many bytes in the match
 }
 ++sliding_window_pointer;
 } 
Then you just have to count how many bytes are equal, or even if we have a real match.
match_length=0;
while(*(current_pointer+match_length)==*(sliding_window_pointer+match_length))
 ++match_length; 
So when you break this loop you have how many bytes are equal, in case match_length is above than 2, then you can directly compress that. Of course there you should care about the end of the file, the length of the sliding window and such things, but this is the beauty of doing a parser, and you should do it.
Possible bugs
If you have any bug, if something crashes, or the decompresed file isn't the same, read this. Of course the first thing is to have the bit Io (input/output) without any bug, check it extensively. Are you sure that you write the offsets the way they should be write? You can have a bug when doing the offset to the start of the sliding window. Another source of bugs is to stop scanning when looking for a matches. May be you don't stop scanning when the file is over. May be you forgot to restore the pointer after a failed match. Are you sure you've write all the bits to the output file? If you ever change the length of any element like the offset or the length remember to change it also in the decompressor. When you read an offset, you remember to get the pointer to the current position and subtract to it the value you've just read?
How to improve it
Well, you've already implemented it, but you need even more compression, or more speed, then read this section.
First take a look at this string: "444 4444 4444" Imagine you are compressing it, and you have already compressed "444 4444" then you read '4' and found a match, "444" right? NO, You have found a match, but maybe it isn't the better one, what you have to do is save temporally this, and continue till you've scanned all the buffer, (till the end, the current position) and then get the better one and write it. The way you can do that is the following: Have a variable called _offset_ and _best_length_, any itineration put _best_length_=0 then scan, once you've found one match compare it's length with _best_length_ if it's above or equal set _best_length_ to the current length and _offset_ to the offset of the match. (phrase) In this case we'll scan the string find "444" and save it as the best match, but continue parsing, and then when we find "4444" then we set it to the best match, because it's length is above than the previous. Once we've finished scanning the whole sliding window you get the best length.
More for the offset: we may do a variable offset length so it depends on the length of the loop back buffer in the current position. Example: If we are in position 412 bytes we need 9 bits to hold that number, so we read only 9 bits, if we are in 19000 we will read 14 bits. For this you may use a instruction called bsr, wich performs log base2 (X). (the number of bits needed to represent a value) (this instruction is very slow and may be emulated, visit www.agner.org if you are interested in so) Also you should think about a finite sliding window, not an infite, well choose what best suits your needs, but remember the bigger the sliding window, the slower the compressor, but also the better the compression is.
More about the offsets: we may do another thing to have variable length offsets, we may use a bit as a flag for saying if we have an offset with the maximum length (let's say 15 bits) or with the minimum length. (9 bits) You should tune this parameters for all kind of files. If you are using the algorithm for just a file (your exe, data or whatever) you should tune this values based on the properties of the file itself. Do the following, compress it, and keep track of how many lengths any offset needed, and also the same for the lengths, and then print them out. And based on this results choose the better values. And save this along with the compressed data, or just recompile both compressor and decompressor, so they use such values.
But even there are more improvements. In fact the way we are compressing is the greedy version. There are optimal ways of compression with the lz family of algorithms, usually they work in the following way: they get the better offsets/lengths for every byte, and once they've computed it for the whole file, they scan it again and choose the best pairs. However this topic will not be covered there, instead I'll talk you about a little step towards optimal parsing: Lazy encoding, the theory behind says that sometimes is better discarding some bytes so you can find better matchs. Example, look this string: "curry urrent current" Once we are in "current" we could choose a match, "curr" at offset 0, but a Lazy parser, will temporally discard "c" and search for more matches, in this case it will find a match of 6 bytes in "urrent". Then it will output the byte "c" and the Code Word (offset/length) of the match. Be careful when discarding bytes with Lazy encoding. If you use this scheme let me know. I've already used it.
The best solution for optimal parsing seems to be Flexible parsing.
Do you still need more? Well, there aren't more compression improvements but using an entropy coder for further compression of the data. In fact you can reduce the literals (raw bytes) to the half and match lengths to 1/4. You can use Static Huffman, or Adaptive Huffman, or Arithmetic Coding, or a Range Coder. However, if you don't want to use an entropy coder, or it's too big for your application or whatever, then you can use variable length codes, then have a look at the section with the same name. An entropy coder may reduce the size of the literals to the half, and the match lengts to 1/4, it's worth the job.
Usually we are using a brute force search method, and this is very slow. So, one big speed improvements is using a binary tree or suffix search tree, it's very fast to acces them. In fact that's what most compressors use. (Look at Mark Nelson's Home page) Another way could be using hashing, lzp uses it to only have to check a few positions.
Another speed improvement is described in the next section.
Tags
Apart from the slow parsing lz77 also has another problem which can be avoided, the bit Io. How do we avoid it? we just make our elements fit in bytes. Think about it, the literal is already a byte, so no problem with it. The offset and length pair, we can adjust them so they are two bytes or three bytes. A very used solution is the following, 12 bits for the offset (the sliding window is 4096 bytes long) and 4 bits for the length. (its range is from 3-19) Then we just have to find any solution for the flags... We'll group them in a byte.
Now we'll keep the number of flags write and the data that they represent. (literals and Code Word) So, when we have computed 8 flags with his data we output a byte with the flags together, and then the data, literals and Code Words which also fit in bytes. Usually one uses an sliding window which a length of 4096, and a match length of four bits, thus saving an offset and a match length in one word.
I never used this method, so if you use it email me, and tell me what data structures you've used for it and the problems
you've found, and also the speed improvements.
Variable length codes
A code is a group of bits which represent a symbol, (value, for example a length) they usually have different lengths. A compressor instead of the symbol it outputs the code, and the decompressor reads the code, and then it gets the symbol associated to it. There are some ways of saving values with codes wich have different lengths, but both of them rely on the fact that values with lower value have more probabilities.
Let's see the first, it's very easy to implement, so it's very useful if you care about the size of your compressor. The idea is the following, put as many 0s as the value is, and end it with a 1. If we encode the value 0 it has the code 0b, if we encode 1, it
will be 01b, if we encode 2 001b, 3 0001b. So you just have to keep track of how many 0s you've read till you read a 1. This is only optimal for a probabilities like the following:
Symbol 
Probability 
0
8/19 
1
5/19 
2
3/19
3
2/19
4
1/19
As you can see those probabilities are the Fibbonaci series, that can annoy a huffman codec, but this is another history, and will be explained in another moment. Also another way of doing variable lengths codes is the following: It tries to use log base2 (symbol) but because it needs a way of telling the decompressor when to stop, it saves the higher value to tell that it should read more bits:
 Symbol 
Code 
0
00 
1
01
2
10 
3
11 000 
4
11 001 
5
11 010
6
11 011
7
11 100 
8
11 101
9
11 110
10
11 111 0000
This one is better than the first, a little bit difficult to implement, but not a lot. I learned it in Charles Bloom's home
page, in the article about lzp. Both those tricks should only be used to represent lengths, and only if we expect that the lengths with lower value are more probable. (they tend to occur more)
Closing words
First af all, if there is any problem with the putbits, don't mind too much, rewrite it, be sure you get and put the bits in a correct way, etc. Now I suppose that you have read the whole text and you still have to start to work, it may seem a little bit difficult, but it isn't, I learned all this stuff in two weeks with the only help of the mentioned articles of the Hugi #12 It was my first compressor, so I had to do the putbits and getbits. If you liked lz77, lzp may be easy to learn.
I should give some thanks to the following people for the help they provided me:
 Charles Bloom Mark Nelson Picard/Rhyme Ross Williams
If you are looking for more info about compression you should have a look at my h-page, http://www.linuxman.2ya.com, where you can find more articles and some useful links.
Contacting the author
If you located any error, or do you think that something could be explained in a better way, email to: virushacker23@yahoo.com See you in next article!
 Jim Reforma, PH 1999-2005
This article comes from Jim Reforma home page at http://www.linuxman.2ya.com Visit again soon for new and updated compression articles and software.
```

