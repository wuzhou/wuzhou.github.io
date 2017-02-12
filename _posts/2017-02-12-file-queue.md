---
author: wuzhou
comments: false
date: 2017-02-12 11:20:00+00:00
layout: post
slug: file-queue
title: 使用文件系统实现队列数据结构
categories:
- programming
tags:
- objective-c
---

队列（Queue）是一种很常用的数据结构。之前我都是在运行内存中使用这种数据结构。但是如果需要一个可以持久化的队列缓存，仅仅是使用内存中的队列数据结构就不行了，因为简单对队列对象进行序列化和反序列化处理，又显的不够高效，所以就考虑到使用文件系统（File System）来实现一个队列数据结构。

# 1. 基本需求

1. 要满足队列最基本的特征：“先进先出”（FIFO）以及一些标准的操作
2. 队列中的元素可以是任意数据类型
3. 可以随时对队列中数据进行读取和写入

# 2. 实现思路

我查阅的一些资料后，发现了一个 [Java 版的实现](https://www.oschina.net/code/snippet_6981_4271) 基本可以满足需求。因为我要在 iOS 中使用这个队列，所以要用 Objective-C 改写一下。在改写完成下面把使用文件系统实现队列的基本思路和要点记录下来，这样以后用其他语言也都能实现了。

先介绍一下文件结构：

队列文件结构：头文件 + 数据段（包含很多个元素）。

元素文件结构：元素头文件 + 元素数据段。

## 2.1 队列文件的头文件

这个头文件是用来描述整个队列，我们需要定义以下这些值：

- 文件长度，用于在新插入一个元素时，判断是否需要增加这个文件的长度
- 元素个数，方便的实现查询队列中元素个数
- 第一个元素数据段开始位置，用于实现 `peek` 操作
- 最后一个元素数据段的开始，用于实现  `Add` 操作

这几个数据都是`int` 类型，这样头文件就是固定长度。在读取文件时，在固定位置就能读到相应的值。

代码：

```
-(void)readFileHeader
{
    [self.fh seekToFileOffset:0];

    NSData *data = [self.fh readDataOfLength:HEADER_LENGTH];

    self.headBuffer = [NSMutableData dataWithData:data];

    self.fileLength = [self readInt:self.headBuffer Offset:0];

    self.elementCount = [self readInt:self.headBuffer Offset:4];

    int firstPos = [self readInt:self.headBuffer Offset:8];
    self.first = [self readElement:firstPos];

    int lastPos  = [self readInt:self.headBuffer Offset:12];
    self.last = [self readElement:lastPos];
}
```

## 2.2 元素头文件

描述元素本身的头文件包括的数据有：

- 元素自身开始的位置
- 元素数据长度

就像这样：

```
typedef struct
{
    int position;
    int length;
}QueueElement;
```

由于头文件长度固定，通过这两个文件就可以读取到这个元素的数据，已及下一元素开始的位置。

## 2.3 实现 Peek 操作

Peek 操作就是读取队列中的第一个元素。由于在队列头文件中保存了第一个元素数据的段的开始位置。再加上元素头文件的信息，我们就可以顺利读出第一个元素。

第一个元素的数据开始的位置 = 第一个元素开始的位置 + 元素头文件长度。

再加上已知数据的长度，这样的就可以顺利读出数据了。如果数据只是增加，也就是文件长度一值在增长，这种读取方式就够了。但是，实际情况是，有可能我们会移除队列中的元素，为能够重复利用空间，新加入的元素可能会利用到已删除元素的空间，所以在读取的时候也要考虑到这种情况，这就是环形读取（ring read）。

### 2.2.1 环形读取

第一种情况：当元素数据段开始位置加上元素数据段长度小于队列文件长度，只需要正常读取。

第二种情况，当元素数据段开始位置加上元素数据段长度大于队列文件长度，这就说明，数据段并没有完全写在文件物理位置的尾端，而是可能还有一部分写在的物理位置的前端，所以在读取时需要考虑到这两部分的数据。

代码：

```
-(NSData *)ringRead:(int)startPos DataLength:(int)dLength
{
    startPos = [self wrapPosition:startPos];

    if (startPos + dLength <= self.fileLength)
    {
        [self.fh seekToFileOffset:startPos];
        NSData *data = [self.fh readDataOfLength:dLength];

        return data;
    }
    else
    {
        int countBeforeEOF = self.fileLength - startPos;
        [self.fh seekToFileOffset:startPos];

        NSData *beforeDataRead = [self.fh readDataOfLength:countBeforeEOF];//读取尾端数据

        NSMutableData *readData = [NSMutableData dataWithData:beforeDataRead];

        int countAfterEOF = dLength - countBeforeEOF;
        [self.fh seekToFileOffset:HEADER_LENGTH];

        NSData *afterDataRead = [self.fh readDataOfLength:countAfterEOF];//读取前端数据

        [readData appendData:afterDataRead];

        return readData;
    }
}
```

## 2.4 实现 Add 操作

向队列中添加一个元素，这个元素需要添加在当前最后一个元素的后面，同样考虑到重复利用空间的问题，我们需要考虑环形写入（ring write）。

### 2.4.1 环形写入

第一种情况：当需要写入的元素数据段开始位置加上元素数据段长度小于队列文件长度，直接从最后一个元素的尾端开始写入就行。

第二种情况：当需要写入的元素数据段开始位置加上元素数据段长度小于队列文件长度，这时就需要考虑当前文件空闲空间的大小，如果空闲空间足够，利用空闲空间进行写入，如果空间不够就把文件长度加大到足够大再进行写入。

代码：

```
-(void)ringWrite:(int)startPos DataToWrite:(NSData *)data
{
    startPos = [self wrapPosition:startPos];

    if (startPos + [data length] <= self.fileLength)
    {
        [self.fh seekToFileOffset:startPos];
        [self.fh writeData:data];
        [self.fh synchronizeFile];
    }
    else
    {
        int countBeforeEOF = self.fileLength - startPos;
        NSData *beforeDataToWrite
                = [data subdataWithRange:NSMakeRange(0, countBeforeEOF)];

        [self.fh seekToFileOffset:startPos];
        [self.fh writeData:beforeDataToWrite];

        int countAfterEOF   = (int)([data length] - countBeforeEOF);
        NSData *afterDataToWrite
                = [data subdataWithRange:NSMakeRange(countBeforeEOF, countAfterEOF)];

        [self.fh seekToFileOffset:HEADER_LENGTH];
        [self.fh writeData:afterDataToWrite];

        [self.fh synchronizeFile];
    }
}
```

在增加文件长度的时候，为保障在写入文件时数据正确，就要使最后一个元素后有足够的连续空间，这样就需要把元素在物理上从前到后进行连续排列。

代码：

```
-(void)expandFileLengthIfNecessary:(int)dataLength
{
    int lengthToWrite = ELEMENT_HEADER_LENGTH + dataLength;

    int remainLength = [self getRemainLengthInFile];

    if (remainLength >= lengthToWrite)
    {
        return;
    }
    else
    {
        NSLog(@"Expand!");
        int previousLength = self.fileLength;
        int newLength;
        do
        {
            remainLength += previousLength;
            newLength = previousLength << 1;
            previousLength = newLength;
        } while (remainLength < lengthToWrite);

        [self.fh truncateFileAtOffset:newLength]; //增加文件长度

        [self.fh seekToFileOffset:0];

        // If the buffer is split, we need to make it contiguous. //保证空间连续性，对空间元素进行物理上的排序
        if(self.last.position < self.first.position)
        {
            [self.fh seekToFileOffset:0];
            NSData *fileData = [self.fh readDataToEndOfFile];

            NSMutableData *newFileData = [NSMutableData dataWithData:fileData];

            int moveLength = (self.last.position + ELEMENT_HEADER_LENGTH + self.last.length) - HEADER_LENGTH;

            NSData *dataToMove = [newFileData subdataWithRange:NSMakeRange(HEADER_LENGTH, moveLength)];

            [newFileData replaceBytesInRange:NSMakeRange(self.fileLength, moveLength) withBytes:[dataToMove bytes]];

            [self.fh writeData:newFileData];

            int newLastElementPos = self.fileLength + self.last.position - HEADER_LENGTH;

            QueueElement newLastElement;
            newLastElement.position = newLastElementPos;
            newLastElement.length = self.last.length;

            self.last = newLastElement;
        }

        self.fileLength = newLength;

        [self writeFileHeader:self.fileLength ElementCount:self.elementCount
                     FirstPos:self.first.position LastPos:self.last.position];
    }
}
```

## 2.5 其他操作

其他操作无非就是对头文件的更新和读取，在这里就不详细描述了，具体代码可以查看项目中的代码。

# 3. 总结
总结下来使用文件系统实现这队列的要点有这么几个：
1. 队列数据结构的特征
2. 头文件的定义
3. 环形读取和写入
4. 保持空间的连续性

# 4. 项目地址

https://github.com/wuzhou/oc-file-queue
