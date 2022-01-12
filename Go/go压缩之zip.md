# archive/zip 实现打包压缩及解压


## 背景知识

ZIP的作者是一个叫Phil Katz的人。Phil Katz这个人是个牛逼程序员，成名于DOS时代，计算机早期网速很慢，拨号使用的是只有几十Kb（比特不是字节）的猫，56Kb实际上是这种猫的最高速度，在ADSL出现之后，这种技术被迅速淘汰。当时记录文件的也是硬盘，但是在电脑之间拷贝文件的是软盘，最高容量记得是1.44MB，这还是200X年的软盘，以前的软盘容量具体多大就不知道了，Phil Katz上网的时候还不到1990年，WWW实际上就没出现，浏览器当然是没有的，当时上网干嘛呢？基本就是类似于网管敲各种命令，这样实际上也可以聊天、上论坛不是吗，传个文件不压缩的话肯定死慢死慢的，所以压缩在那个时代很重要。当时有个商业公司提供了一种称为ARC的压缩软件，可以让你在那个时代聊天更快，当然是要付费的，Phil Katz就感觉到不爽，于是写了一个PKARC，免费的，看名字知道是兼容ARC的，于是网友都用PKARC了，ARC那个公司自然就不爽，把哥们告上了法庭，说牵涉了知识产权等等，结果Phil Katz坐牢了。。。牛人就是牛人， 在牢里面冥思苦想，决定整一个超越ARC的牛逼算法出来，牢里面就是适合思考，用了两周就整出来的，称为PKZIP，不仅免费，而且这次还开源了，直接公布源代码，因为算法都不一样了，也就不涉及到知识产权了，于是ZIP流行开来，不过Phil Katz这个人没有从里面赚到一分钱，还是穷困潦倒，因为喝酒过多等众多原因，2000年的时候死在一个汽车旅馆里。英雄逝去，精神永存，现在我们用UE打开ZIP文件，我们能看到开头的两个字节就是PK两个字符的ASCII码。

## 压缩案例

和 tar 的过程很像，只有些小的差别。

```
package main

import (
    "archive/zip"
    "fmt"
    "io"
    "log"
    "os"
    "path/filepath"
    "strings"
)

func main() {

    var src = "log"
    var dst = "log.zip"

    if err := Zip(dst, src); err != nil {
        log.Fatalln(err)
    }
}

func Zip(dst, src string) (err error) {
    fw, err := os.Create(dst)
    defer fw.Close()
    if err != nil {
        return err
    }

    zw := zip.NewWriter(fw)
    defer zw.Close()

  
    return filepath.Walk(src, func(path string, fi os.FileInfo, errBack error) (err error) {
        if errBack != nil {
            return errBack
        }

     
        fh, err := zip.FileInfoHeader(fi)
        if err != nil {
            return
        }

       
        fh.Name = strings.TrimPrefix(path, string(filepath.Separator))  // 重要步骤

       
        if fi.IsDir() {
            fh.Name += "/"  //重要步骤 有小坑
        }

        w, err := zw.CreateHeader(fh)
        if err != nil {
            return
        }

      
        if !fh.Mode().IsRegular() {  //常规文件才继续写入
            return nil    
        }

      
        fr, err := os.Open(path)
        defer fr.Close()
        if err != nil {
            return
        }

       
        n, err := io.Copy(w, fr)  //内存拷贝 
        if err != nil {
            return
        }
     

        return nil
    })
}
```

## 解压缩案例

```

package main

import (
    "archive/zip"
    "fmt"
    "io"
    "log"
    "os"
    "path/filepath"
)

func main() {
    var src = "log.zip"
 
    var dst = "./"

    if err := UnZip(dst, src); err != nil {
        log.Fatalln(err)
    }
}

func UnZip(dst, src string) (err error) {
    
    zr, err := zip.OpenReader(src)
    defer zr.Close()
    if err != nil {
        return
    }
     _, err := os.Stat(dst) //判断目录是否存在
     
    if err != nil {
        if err := os.MkdirAll(dst, 0755); err != nil {
            return err
        }
    }

    // 遍历 zr ，将文件写入到磁盘
    for _, file := range zr.File {
        path := filepath.Join(dst, file.Name)

   
        if file.FileInfo().IsDir() {
            if err := os.MkdirAll(path, file.Mode()); err != nil { //如果是目录只需创建文件夹 不需要写入数据
                return err
            }
            continue
        }

    
        fr, err := file.Open()
        defer  fr.Close()
        if err != nil {
            return err
        }

        fw, err := os.OpenFile(path, os.O_CREATE|os.O_RDWR|os.O_TRUNC, file.Mode())
        defer  fw.Close()
        if err != nil {
            return err
        }

        n, err := io.Copy(fw, fr)
        if err != nil {
            return err
        }
      
    }
    return nil
}

```

## 常见坑

使用我们第一个Zip函数压缩的文件，如果传入的是文件而不是文件夹，例如window下 srcFile是 C:\src\1.zip。
这样用Unzip函数无法解压，比如destDir路径设置为E:\UnzipFile

header.Name += "/" 这句代码直接使用  /  不太好

### 解决办法

````
//header.Name = strings.TrimPrefix(path, filepath.Dir(srcFile) + "/") //原来
header.Name = strings.TrimPrefix(path, srcFile + string(os.PathSeparator)) //修复
```

## 不包含目录文件夹的情况下将文件压缩为 .zip？

场景 dst 不带目录直接是一个文件例如 bak.zip

### 解决办法

只需在 zip header 中使用文件的基本名称即可。

```
header.Name = filepath.Base(filename)

```