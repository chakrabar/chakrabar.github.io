---
layout: default
title: "Comparison of serialization in .Net"
date: 2017-11-06
tags: tech tips notes
---



# A brief comparison of available serialization techniques 

## Experience and data from practical use in enterprise applications

Here we'll see a comparative study of **file size** and **processing time** to serialize some typical data to different serialization formats, and to compress (zip) the same. 

### Serializations techniques that we'll see are:
1. [XML](https://en.wikipedia.org/wiki/XML)
2. [JSON](https://www.json.org/)
3. [Binary Formatter](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.formatters.binary.binaryformatter(v=vs.110).aspx)
4. [Protocol Buffers](https://developers.google.com/protocol-buffers/)

Please note this is the default compression ratio used in **.Net [SharpZipLib](https://icsharpcode.github.io/SharpZipLib/)**. The compression factor can be changed and the size/performance will based vary on that.


| |Size of data(KB)|Serialization time(ms)|Zipped file size(KB)|Compression time(ms)|
|---|---:|---:|---:|---:|
JSON|12460|1100|205|1200
XML|19755|600|259|2000
BinFor|8209|770	|1064|3800
ProtBuf|5347|155|205|620

### Now, before making any judgement, here are few important points to consider:

NOTE #1: Not all serializations are equally **_code friendly_**. What I mean here, some of them does not need any changes in the actual class for them to work, while some of them needs lot of work. 

- JSON: No customization needed as such
- XML: Based on the nature of data and desired format of output XML, it may need some XML specific annotations. Also, there are constraints like, Dictionary<TKey, TData> doesn't work out of the box.
- Protocol Buffer: It's mandatory to decorate the class and each property with annotation and ProtoMember unique integer.
- Binary Formatter: All the classes involved, needs a Serializable attribute.

NOTE #2: Consider how **_platform agnostic_** they are how easily usable across all major technology stack.
- XML is pretty much platform independent.
- JSON is also similarly platform independent. Also, JSON being JavaScript native, supported in all browsers and web.
- ProtoBuf also has pretty good platform support, though I didn't test it cross-platform myself.
- Binary Formatting works only with .Net. Though now .Net is making its way through all major platforms (Windows, Linux, MacOS) with .NetCore, I'm not sure how much Binary Formatter is supported.

NOTE #3: **Readability**   
`JSON > XML > Binary Format, Protobuf (not human readable)`

Conclusion (IMHO)

> So, we can see **ProtoBuf** wins easily on performance. But a big overhead is attribute annotation needed for each property of each class that needs to be serialized.

> On the other hand **JSON**, though not the most performant, wins on ease of use, readability & wide platform acceptance.

I'll be posting sample code that I used for all the serialization and file compression soon.

	