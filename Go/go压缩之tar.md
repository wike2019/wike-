# archive/tar 实现打包压缩及解压

## 主要功能

这个包比较简单，就是将文件或目录进行打包和解包。因为很多压缩算法不支持目录压缩，所以我们一般先使用tar包把目录打包成一个文件，在使用其他压缩算进一步处理。
例如gzip

## 使用场景

数据备份，例如日志啊，一些文字资源，定时备份到远程，可以先tar打包，再压缩，减小体积，从而节约成本


![image](https://csdn.52wike.com/wike_blog/2022-01-11/64922ef2-ab2f-4c11-83f6-4bf55e75590f.png)

(说明：图片来自维基百科）
 
## 单个文件操作

```
package main

import (
    "os"
    "log"
    "archive/tar"
    "fmt"
    "io"
)

func main() {
    // 准备打包的源文件
    var srcFile = "wike"
    // 打包后的文件
    var desFile = fmt.Sprintf("%s.tar",srcFile)

  
    fw, err := os.Create(desFile) //创建目标文件
    log.Println(err)
    defer fw.Close()
    
    tw := tar.NewWriter(fw)   // 通过 fw 创建一个 tar.Writer
   
    defer tw.Close()  // 这里不要忘记关闭，如果不能成功关闭会造成 tar 包不完整

    
    fi, err := os.Stat(srcFile)  //获取文件描述句柄   
    log.Println(err)
    hdr, err := tar.FileInfoHeader(fi, "")  //通过文件句柄生成文件头信息

    err = tw.WriteHeader(hdr) // 将 tar 的文件头信息 hdr 写入到 tw
    log.Println(err)

  
    fr, err := os.Open(srcFile)   // 打开准备写入的文件
    log.Println(err)
    defer fr.Close()

    _, err := io.Copy(tw, fr) //全量数据拷贝,大文件容易出现问题，可以通过其他手段优化
    log.Println(err)
}

```

## 单个文件解包

```

package main

import (
    "os"
    "archive/tar"
    "io"
    "log"
)

func main() {

    var srcFile = "wike.tar"

  
    fr, err := os.Open(srcFile)  // 将 tar 包打开
    
    log.Println(err)
    
    defer fr.Close()

    tr := tar.NewReader(fr) // 通过文件句柄fr，创建一个可读流
 
    for hdr, err := tr.Next(); err != io.EOF; hdr, err = tr.Next(){
        if err!=nil{
          log.Println(err) //处理错误，一般不会发生
        }
    
   
        fi := hdr.FileInfo()      // 获取文件信息

       
        fw, err := os.Create(fi.Name())  // 创建一个空文件，用来写入解包后的数据
        defer  fw.Close()
        log.Println(err)

      
        n, err := io.Copy(fw, tr) //全量数据拷贝,大文件容易出现问题，可以通过其他手段优化
        log.Println(err)
       
    }
}
````

## 目录打包(重要）



```
package main

import (
    "archive/tar"
    "compress/gzip"
    "fmt"
    "io"
    "log"
    "os"
    "path/filepath"
    "strings"
)

func main() {
    // 修改日志格式，显示出错代码的所在行，方便调试，实际项目中一般不记录这个。

    var src = "apt"
    var dst = fmt.Sprintf("%s.tar", src)

    // 将步骤写入了一个函数中，这样处理错误方便一些
    Tar(src, dst); 
}

func Tar(src, dst string)  {
    // 创建文件
    fw, err := os.Create(dst)
    if err != nil {
        return
    }
    defer fw.Close()
    tw := tar.NewWriter(gw)
    defer tw.Close()
    
    s, err := os.Stat(src) 
    
    if !s.IsDir(){
        return //不是目录不处理
    } 
    
    
    //filepath.Walk这个函数很重要遍历目录文件

    filepath.Walk(src, func(fileName string, fi os.FileInfo, err error) error {
    
        if err != nil { 
            return err     // 处理错误一般不会出错
        }

     
        hdr, err := tar.FileInfoHeader(fi, "")
        if err != nil {
            return err
        }

        hdr.Name = strings.TrimPrefix(fileName, string(filepath.Separator))

  
        if err := tw.WriteHeader(hdr); err != nil {
            return err
        }

       
        if !fi.Mode().IsRegular() {
            return nil
        }

        fr, err := os.Open(fileName)
        defer fr.Close()
        if err != nil {
            return err
        }

        n, err := io.Copy(tw, fr)
        if err != nil {
            return err
        }


        return nil
    })
}
```

### 目录解包解压


```

func UnTar(dst, src string)  {
  
    fr, err := os.Open(src)  
    if err != nil {
        return
    }
    defer fr.Close()

    tr := tar.NewReader(fr)

 
    for {
        hdr, err := tr.Next()

        switch {
        case err == io.EOF:
            return nil
        case err != nil:
        
            return err
            
        case hdr == nil:
            
            continue
        }

    
        dstFileDir := filepath.Join(dst, hdr.Name)

        switch hdr.Typeflag {
        
            case tar.TypeDir: 
                if b := ExistDir(dstFileDir); !b {
                       
                    if err := os.MkdirAll(dstFileDir, 0775); err != nil {
                        return err
                    }
                }
            case tar.TypeReg: 
             
                file, err := os.OpenFile(dstFileDir, os.O_CREATE|os.O_RDWR, os.FileMode(hdr.Mode))
                defer file.Close()
                if err != nil {
                    return err
                }
                _, err := io.Copy(file, tr)
                if err != nil {
                    return err
                }
            }
    }

}

```

### 实现流程

打包和解包的原理和实现
1、打包实现原理
先创建一个文件xxx.tar，然后向xxx.tar写入tar头部信息。打开要被tar的文件，向xxx.tar写入头部信息，然后向xxx.tar写入文件信息。重复第二步直到所有文件都被写入到x.tar中，关闭x.tar，整个过程就这样完成了

2、解包实现原理
先打开tar文件，然后从这个tar头部中循环读取存储在这个归档文件内的文件头信息，从这个文件头里读取文件名，以这个文件名创建文件，然后向这个文件里写入数据

