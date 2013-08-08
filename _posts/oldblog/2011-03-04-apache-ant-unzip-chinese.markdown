---
layout: post
title: 用apache的ant解压zip文件（中文文件或文件夹解决方法）
date: 2013-07-26 12:01:00
description: 用apache的ant解压zip文件（中文文件或文件夹解决方法）
keywords: ant,解压
---

昨天在研究用apache的ant包来解压zip文件，把经验写下来与大家分享。
写贴上代码
{% highlight java %}
package upzip;  
  
import java.io.File;  
import java.io.FileOutputStream;  
import java.io.InputStream;  
import org.apache.tools.zip.*;  
import java.util.Enumeration;  
  
/** 
 * 压缩解压ZIP文件 
 * @author Administrator 
 * 
 */  
public class AntZip {  
    private ZipFile zipFile;  
    private static int bufSize;    //size of bytes  
    private byte[] buf;  
    private int readedBytes;  
      
    /** 
     *  
     * @param bufSize 缓存大小 
     */  
    public AntZip(int bufSize){  
        this.bufSize = bufSize;  
        this.buf = new byte[this.bufSize];  
    }  
      
    public AntZip(){  
        this(1024);  
    }  
      
    /** 
     * 生存目录 
     * @param directory 解压文件存放目录 
     * @param subDirectory 子目录（没有时可传入空字符串） 
     */  
    private void createDirectory(String directory, String subDirectory) {  
        String dir[];  
        File fl = new File(directory);  
        try {  
            if(subDirectory == "" && fl.exists() != true)  
                fl.mkdir();  
            else if(subDirectory != ""){  
                dir = subDirectory.replace('\\', '/'').split("/");  
                for (int i = 0; i < dir.length; i++) {  
                    File subFile = new File(directory + File.separator + dir[i]);  
                    if (subFile.exists() == false)  
                        subFile.mkdir();  
                    directory += File.separator + dir[i];  
                }  
            }  
        }catch (Exception e){  
            e.printStackTrace();  
        }  
    }  
      
    /** 
     * 解压指定的ZIP文件 
     * @param unZipFileName 文件名字符串（包含路径） 
     * @param outputDirectory 解压后存放目录 
     */  
    public void unZip(String unZipFileName,String outputDirectory){  
        FileOutputStream fileOut;     
        //File file;     
        InputStream inputStream;  
          
        try{  
            createDirectory(outputDirectory,"");  
            if(System.getProperty("os.name").toLowerCase().indexOf("windows") >= 0){  
                this.zipFile = new ZipFile(unZipFileName,"GBK");  
            }else if(System.getProperty("os.name").toLowerCase().indexOf("linux") >= 0){  
                this.zipFile = new ZipFile(unZipFileName,"UTF-8");  
            }  
            for(Enumeration entries = this.zipFile.getEntries();entries.hasMoreElements();){  
                ZipEntry entry = (ZipEntry)entries.nextElement();  
                if(entry.isDirectory()){  
                    //是目录，则创建之  
                    String name = entry.getName().substring(0, entry.getName().length() - 1);  
                    File f = new File(outputDirectory + File.separator + name);  
                    f.mkdir();  
                    //file.mkdirs();  
                }else{  
                    //是文件  
                    String fileName = entry.getName().replace('\\', '/'');  
                    if (fileName.indexOf("/") != -1){  
                        createDirectory(outputDirectory,fileName.substring(0, fileName.lastIndexOf("/")));  
                        fileName=fileName.substring(fileName.lastIndexOf("/") + 1,fileName.length());  
                    }  
                    File f = new File(outputDirectory + File.separator + entry.getName());  
                    f.createNewFile();  
                    inputStream = zipFile.getInputStream(entry);  
                    fileOut = new FileOutputStream(f);  
                    while(( this.readedBytes = inputStream.read(this.buf) ) > 0){  
                        fileOut.write(this.buf , 0 , this.readedBytes );  
                    }  
                    fileOut.close();  
                    inputStream.close();  
                }  
            }  
            this.zipFile.close();  
        }catch(Exception e){  
            e.printStackTrace();  
        }  
    }  
      
    /** 
     * 解压指定ZIP文件 
     * @param unZipFile 需要解压的ZIP文件 
     */  
    public void unZip(File unZipFile){  
        String outputDirectory = new String("D:/ip/"); //解压后存放目录  
         unZip(unZipFile.toString(),outputDirectory);  
    }  
}
{% endhighlight %}
遇到的做大问题就是中文文件名。原以为ant支持中文，所以没问题，但实际做下来，发现中文文件名会出现乱码。在网上找了好久，发现原来是和系统有关。中文windows系统默认的字符集是GBK，而ant就不知道用什么字符集在读取文件名了（估计是JAVA默认的字符集）。所以导致了文件名乱码。解决的方法就是ZipFile(unZipFileName,"GBK")，后面加个字符集，可以指定用什么字符集解压zip。Linux下有个问题，Linux系统默认的字符集是UTF-8，可以参照windows下的，用UTF-8做解压的字符集。 

{% highlight java %}
if(System.getProperty("os.name").toLowerCase().indexOf("windows") >= 0){  
    this.zipFile = new ZipFile(unZipFileName,"GBK");  
}else if(System.getProperty("os.name").toLowerCase().indexOf("linux") >= 0){  
    this.zipFile = new ZipFile(unZipFileName,"UTF-8");  
}
{% endhighlight %}
没找到统一的决绝方法，待研究。
在Linux下还有问题，就是从windows系统中拷贝过去的文件带中文名的，在linux下应该使用GBK字符集读取，然后转码成UTF-8。不过，的确Linux下对中文的支持太有问题了。

转码代码[参考地址](http://jlins.iteye.com/blog/574670)

但是如果出现中英文一起的，那就会出现乱码，这个问题没自己看，可能转码代码还不过好。

问题这么多，最后也很无奈，想了个最简单的办法，就是转码后用正则表达式判断是否有中文，如果有，直接换个英文文件名，一切OK。 
