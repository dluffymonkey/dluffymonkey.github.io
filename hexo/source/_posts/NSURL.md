#### [iOS-Foundation] NSURL

[NSURL /NSURLComponents](https://link.jianshu.com/?t=http://nshipster.com/nsurl/)

#### URI

URL(统一资源定位符) 是一种 URI，URN(统一资源名称) 也是一种 URI，所以 [URI](https://link.jianshu.com/?t=https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6) (统一资源标志符)可被视为定位符，名称或两者兼备。URN 如同一个人的名称，而 URL 代表一个人的住址。换言之，URN 定义某事物的身份，而 URL 提供查找该事物的方法。例如，ISBN 0-486-27557-4 无二义性地标识出莎士比亚的戏剧《罗密欧与朱丽叶》的某一特定版本，它可以允许人们在不指出其位置和获得方式的情况下谈论这本书。为获得该资源并阅读该书，人们需要它的位置，也就是一个URL地址。在类Unix操作系统中，一个典型的URL地址可能是一个文件目录，例如
file:///home/username/RomeoAndJuliet.pdf，该URL标识出存储于本地硬盘中的电子书文件。因此，URL和URN有着互补的作用。

![img](https://upload-images.jianshu.io/upload_images/669609-d7c197fce85aac99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/180)

#### URL Encode

URL 采用 ASCII 编码格式，所以不支持如中文等非 ASCII 码字符，另外 URL 中保留的分隔符号(`?`、`&`、`=`等)也无法作为内容，否则会引起歧义。这就需要通过编码，用安全的字符来表示这些不符合要求的字符，格式是`%`加两位安全字符，所以 URL 编码也称为百分号编码。在 iOS 中的相关操作分别为：

##### Encode

```
NSString *encodedString = [URLString stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];

@interface NSCharacterSet (NSURLUtilities)
+ (NSCharacterSet *)URLUserAllowedCharacterSet;
+ (NSCharacterSet *)URLPasswordAllowedCharacterSet;
+ (NSCharacterSet *)URLHostAllowedCharacterSet;
+ (NSCharacterSet *)URLPathAllowedCharacterSet;
+ (NSCharacterSet *)URLQueryAllowedCharacterSet;
+ (NSCharacterSet *)URLFragmentAllowedCharacterSet;
@end

```

##### Unencode

```
+ (NSString *)decodeURLString:(NSString *)URLString {
    // 有时从服务端获取的 URL 中，空格被编码为+, 
    // 而方法- stringByRemovingPercentEncoding只替换百分号编码，
    // 所以要在执行该方法前，先将`+`替换掉(真正的加号字符是被百分号编码的)
    NSString *result = [URLString stringByReplacingOccurrencesOfString:@"+" withString:@" "];
    result = [result stringByRemovingPercentEncoding];
    
    return result;
}

```

#### NSURL

[NSURL](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurl?language=objc)

一个 NSURL 对象代表了一个表示远程服务器资源或者本地文件的 URL。所以在 iOS 的网络请求和操作文件系统的 API 中，很多都需要 NSURL 类型的参数。

##### 创建 NSURL 对象

```
+ URLWithString:
- initWithString:

```

传入的字符串类型的 URL 参数要求是经过编码后的，如果该参数不符合规范或者传入了 nil，那么上述方法会返回 nil。

```
+ URLWithString:relativeToURL:
- initWithString:relativeToURL:

```

除了字符串类型的 URL 参数，还需要一个 NSURL 类型的 Base URL 参数，字符串类型的 URL 将作为 Base URL 的相对路径。同样字符串类型的 URL 参数要求是经过编码后的，如果不符合规范或者传入了 nil，那么也会返回 nil。

```
NSURL *baseURL = [NSURL URLWithString:@"http://example.com/v1/"];
// http://example.com/v1/foo
[NSURL URLWithString:@"foo" relativeToURL:baseURL];
// 如果相对路径以/开始，根据规则，URL 最终会拼装为http://example.com/foo
[NSURL URLWithString:@"/foo" relativeToURL:baseURL];
// http://example2.com/，因为 URLString 并不是相对的 path 部分，而是完整的 URL，所以最终的值会忽略掉 baseURL。
[NSURL URLWithString:@"http://example2.com/" relativeToURL:baseURL];

```

```
+ fileURLWithPath:
- initFileURLWithPath:
+ fileURLWithPath:isDirectory:
- initFileURLWithPath:isDirectory:

```

创建一个 file URL (协议为`file:///`)，参数 Path 需要传入一个系统路径，如果路径中包含~，那么需要先调用 NSString 的`stringByExpandingTildeInPath`方法解析~，如果传入的路径是相对路径，那么生成的 URL 将相对于当前路径。当没有 isDirectory 参数时，如果路径以/结束，则认为路径是一个目录，若不是以/结束，则系统会先检查路径是否存在，若路径存在且是一个目录，则系统会在路径后添加/，若路径不存在，系统则认为路径代表一个文件。而 isDirectory 参数则明确指明路径是否代表一个目录。

```
+ fileURLWithPathComponents:

```

传入一个字符串类型元素的数组，该方法用/将这些元素组装成一个 file URL。

```
+ fileURLWithFileSystemRepresentation:isDirectory:relativeToURL:
- initFileURLWithFileSystemRepresentation:isDirectory:relativeToURL:

```

通过 c string Path 创建文件 URL 对象。

```
- getFileSystemRepresentation:maxLength:

```

将文件 URL 以 c string 填入到提供的 buffer 中。

##### 语法

URI语法由URI协议名，一个冒号，和协议对应的内容所构成。特定的协议定义了协议内容的语法和语义，而所有的协议都必须遵循一定的URI语法通用规则，亦即为某些专门目的保留部分特殊字符。URI语法同时也就各种原因对协议内容加以其他的限制，例如，保证各种分层协议之间的协同性。百分号编码也为URI提供附加信息。

一般的 URI 格式为：

```
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]

```

具体的例子如：

```
                    hierarchical part
        ┌───────────────────┴─────────────────────┐
                    authority               path
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
scheme  user information     host     port                  query         fragment

  urn:example:mammal:monotreme:echidna
  └┬┘ └────────────┬───────────────┘
scheme              path

```

NSURL 根据 [RFC 2396](https://link.jianshu.com/?t=http://www.ietf.org/rfc/rfc2396.txt) 的定义提供了只读属性用于获取 URL 每一部分的值：

- scheme，协议名，不带有连接内容的冒号，如“http”。
- user，解码后的用户名。
- password，解码后的 password。
- host，主机域名或 IP 地址。
- port ，端口号。
- path，authority(user，password，host，port)后的部分，如果 URL 对象是 file URL，该属性会去掉最后的/。
- query，解码后的 query 字符串，没有?，但包括连接多个键值对的&。
- fragment，解码后的片段，不包括#。
- parameterString，path 后由分号分隔的部分，如`file:///path/to/file;foo`中的 foo。
- lastPathComponent，path 部分的最后一段。
- pathExtension，path 指向资源的扩展名。
- pathComponents，由解码后的 path 各段组成的数组。
- resourceSpecifier，冒号后的全部内容。
- absoluteString，将 URL string 和相对的 Base URL 拼接成的完整 URL string。
- absoluteURL，返回一个 NSURL 对象，将 absoluteString 作为该 URL 对象的 string。
- baseURL，如果 URL 对象本身由完整 URL 字符串构成，那么该属性为 nil。
- relativeString，URL 对象的相对部分，即创建时传入的字符串部分。
- relativePath，只包含相对字符串中的 path，而不包括 Base URL 中的。
- standardizedURL，删除掉 path 中的.和..的新的 URL 对象。
- fileSystemRepresentation，c string 表示的文件资源路径。

##### 相等比较

NSURL 重写了`- isEqual:`方法，判断条件是，当两个 NSURL 对象的 baseURL 和 relativeString 都相同时，就认为它们是相等的。

##### 修改和转换文件 URL

```
- fileReferenceURL

```

如果 URL 对象代表了一个 file path URL，那么该方法则返回一个其指向资源对应的 file reference URL，格式如`file:///.file/id=16777217.83600349/` file reference URL 直接引用资源，而不是通过资源所在文件系统的路径，这样即使资源的 path 改变，file reference URL 仍然是有效的。如果 URL 对象不是一个 file path URL 或者 path 指向的资源并不存在，那么该方法会返回 nil。如果 URL 对象本身就是 file reference URL 那么该方法返回对象本身。不要直接持久化保存 file reference URL，因为当系统重新启动后它们不再有效，而应该使用 [bookmark](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurl/1417795-bookmarkdatawithoptions?language=objc)。

```
filePathURL

```

如果 URL 对象是一个 file path URL 且指向的资源确实存在，那么该属性返回对象本身，如果 URL 对象是一个 file reference URL，那么则返回对应的 file path URL，否则返回 nil。

```
- URLByAppendingPathComponent:

```

返回一个新的 file path URL，这个 URL 在原 URL 对象后添加了传入的一段路径参数。该方法会自动在原 URL 对象和新的 path 间加入/，如果 path 参数最后没有以/结尾，那么该方法会同步检测新 URL 对象指向的资源是否存在且是一个目录，如果路径指向网络挂载卷，这将会造成严重的性能损耗。

```
- URLByAppendingPathComponent:isDirectory:

```

功能同上一个方法，不过可以通过 isDirectory 参数具体指明新的 file URL 是否为一个目录，从而避免系统的同步检测。

```
- URLByAppendingPathExtension:

```

返回一个添加扩展名的 URL 对象，该方法会自动去掉原 URL 对象最后的`/`并添加.

```
URLByDeletingLastPathComponent

```

去掉最后一段 path 的 URL 对象

```
URLByDeletingPathExtension

```

去掉最后一段文件扩展名的 URL 对象

```
URLByResolvingSymlinksInPath

```

一个解析了 symbolic link 后的 URL

```
URLByStandardizingPath

```

将原 URL 对象表示的 file path 标准化，包括转换~标识符，去掉多余的空格和/，转换..等操作。

##### file URL 检测

- `fileURL`，布尔值，指明 URL 是否使用`file:///`协议，无论是 file path URL 还是 file reference URL
- `- isFileReferenceURL`，返回布尔值，说明 URL 对象是否为 file reference URL
- `- checkResourceIsReachableAndReturnError:`，返回布尔值，说明 file URL 指向的文件资源是否可获取，若返回 NO，通过 error 参数可以获取更多信息

##### File System Resource Properties

当 URL 对象为 file URL 时，可以通过特定的 key 设置、获取及删除 URL 对应的某些资源。

系统内置的 key ：

[Common File System Resource Keys](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/index.html#//apple_ref/doc/constant_group/Common_File_System_Resource_Keys)
[File Property Keys](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/index.html#//apple_ref/doc/constant_group/File_Property_Keys)
[Ubiquitous Item Property Keys](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/index.html#//apple_ref/doc/constant_group/Ubiquitous_Item_Property_Keys)
[Volume Property Keys](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/index.html#//apple_ref/doc/constant_group/Volume_Property_Keys)

相应的方法：

```
// 获取指定 key 的资源内容，
- getResourceValue:forKey:error:
// 返回一个字典，包含了所有指定 key 的对应资源
- resourceValuesForKeys:error:
// 设置某一资源值，并直接持久化保存
- setResourceValue:forKey:error:
// 传入一个字典，设置多个资源值，持久化保存
- setResourceValues:error:
// 设置自定义资源 key 的值到内存中
- setTemporaryResourceValue:forKey:
// 删除缓存中所有资源 key 的值
- removeAllCachedResourceValues
// 删除缓存中指定资源 key 的值
- removeCachedResourceValueForKey:

```

##### Bookmarks

通过 NSURL 的 bookmarks 相关 API，可以持久化保存文件的路径。所谓 bookmark，是一种 `NSData` 类型的数据，描述了文件路径的信息。将 bookmark 持久化保存，当应用重新启动后，可通过 bookmark 获取到对应文件路径的 URL，即使期间文件被重新命名或移动到其他路径。

```
- bookmarkDataWithOptions:includingResourceValuesForKeys:relativeToURL:error:

```

创建一个 NSURL 对象对应的 bookmark。
参数 options 有两个值，`NSURLBookmarkCreationMinimalBookmark`指通过尽可能少的信息创建一个 bookmark，`NSURLBookmarkCreationSuitableForBookmarkFile`指创建的 bookmark 需要包含用于创建 Finder alias files 的属性。
参数 keys，通过传入 key 的数组，指明哪些 URL 资源需要保存到 bookmark 中
参数 relativeURL 指 bookmark 相对的 URL

```
+ resourceValuesForKeys:fromBookmarkData:

```

获取保存在 bookmark 中的 URL 资源。

```
+ URLByResolvingBookmarkData:options:relativeToURL:bookmarkDataIsStale:error:
- initByResolvingBookmarkData:options:relativeToURL:bookmarkDataIsStale:error:

```

通过解析 bookmark 转换成一个 NSURL 对象，额外的参数：
options，`NSURLBookmarkResolutionWithoutUI`、`NSURLBookmarkResolutionWithoutMounting`
relativeURL，bookmark 相对的 URL
isStale，指定转换后，bookmark 是否失效

```
+ writeBookmarkData:toURL:options:error:

```

指定一个 alias file 的路径，将 NSData 类型的 bookmark 持久化保存在其中，可以将这个 alias file 理解为一个 symbolic link。

```
+ bookmarkDataWithContentsOfURL:error:

```

从创建的 alias file 获取 bookmark

```
+ URLByResolvingAliasFileAtURL:options:error:

```

直接通过 alias file 的 URL 获取保存在其中的 bookmark 对应的真正 URL，options 参数指明了如何解析 bookmark。

#### NSURLComponents

[NSURLComponents](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents?language=objc)

NSURLComponents 是一个用于解析和构建 URL 的类，
通过 NSURLComponents 可以单独修改 URL 的某一部分，获取某一部分编码后的值。

##### 创建一个 NSURLComponents 实例

```
// 通过传入的 URL string 创建
+ componentsWithString:
// 通过传入的 URL 对象创建，resolve 参数决定是否解析 URL 对象的 baseURL
+ componentsWithURL:resolvingAgainstBaseURL:
- init
- initWithString:
- initWithURL:resolvingAgainstBaseURL:

```

##### 获取 NSURL 实例

- `string`，返回 URL string
- `URL`，返回 URL 对象
- `- URLRelativeToURL:`，返回 URL 对象并相对于传入的 baseURL 参数

##### 可读写的 URL 各部分的属性

[fragment](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1417638-fragment?language=objc)
[host](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1411178-host?language=objc)
[password](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1415604-password?language=objc)
[path](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1409650-path?language=objc)
[port](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1413451-port?language=objc)
[query](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1415452-query?language=objc)
[queryItems](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1407752-queryitems?language=objc)
[scheme](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1407517-scheme?language=objc)
[user](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1415026-user?language=objc)

其中`queryItems`属性是一个包含 NSURLQueryItem 对象的数组，每一个 NSURLQueryItem 对象代表了 query string 中的一个键值对

##### 百分号编码后的各部分的读写属性

[percentEncodedFragment](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1418392-percentencodedfragment?language=objc)
[percentEncodedHost](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1418231-percentencodedhost?language=objc)
[percentEncodedPassword](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1410319-percentencodedpassword?language=objc)
[percentEncodedPath](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1408161-percentencodedpath?language=objc)
[percentEncodedQuery](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1410395-percentencodedquery?language=objc)
[percentEncodedUser](https://link.jianshu.com/?t=https://developer.apple.com/reference/foundation/nsurlcomponents/1417767-percentencodeduser?language=objc)