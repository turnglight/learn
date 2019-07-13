# IO

## InputStream

```java
public abstract int read() throws IOException;//读取一个字符
public int read(byte b[]) throws IOException;//读取一个字节数组
public int read(byte b[], int off, int len) throws IOException;//在b数据的off偏移位置读取																	//len个字符
public boolean markSupported();//是否支持标记
public synchronized void reset() throws IOException ;//返回到标记位置
public synchronized void mark(int readlimit);//标记
public void close() throws IOException ;//关闭输入流
public int available() throws IOException;//是否有效
public long skip(long n) throws IOException;//跳过n个字符
```

### FileInputStream

~~~java
public FileInputStream(String name) throws FileNotFoundException ;
public FileInputStream(File file) throws FileNotFoundException;
public FileInputStream(FileDescriptor fdObj);
private native void open0(String name) throws FileNotFoundException;
private void open(String name) throws FileNotFoundException ;
public int read() throws IOException;
private native int read0() throws IOException;
private native int readBytes(byte b[], int off, int len) throws IOException;
public int read(byte b[]) throws IOException ;
public int read(byte b[]) throws IOException;
public int read(byte b[], int off, int len) throws IOException;
public long skip(long n) throws IOException ;
private native long skip0(long n) throws IOException;
public int available() throws IOException;
private native int available0() throws IOException;
public void close() throws IOException ;
public final FileDescriptor getFD() throws IOException ;
public FileChannel getChannel();
private static native void initIDs();
private native void close0() throws IOException;
protected void finalize() throws IOException ;//对象没有引用后，最终保证文件流关闭
~~~

##### **FileDescriptor**

- FileDescriptor 可以被用来表示开放文件、开放套接字等
  - in  -- 标准输入(键盘)的描述符
  - out -- 标准输出(屏幕)的描述符
  - err -- 标准错误输出(屏幕)的描述符

##### **FileChannel**

- FileChannel是一个用读写，映射和操作一个文件的通道。出了读写操作之外，还有裁剪特定大小文件truncate()，强制在内存中的数据刷新到硬盘中去force()，对通道上锁lock()等功能。

  ~~~java
  /**
       * 普通的文件复制方法
       *
       * @param fromFile 源文件
       * @param toFile   目标文件
       * @throws FileNotFoundException 未找到文件异常
       */
      public void fileCopyNormal(File fromFile, File toFile) throws FileNotFoundException {
          InputStream inputStream = null;
          OutputStream outputStream = null;
          try {
              inputStream = new BufferedInputStream(new FileInputStream(fromFile));
              outputStream = new BufferedOutputStream(new FileOutputStream(toFile));
              byte[] bytes = new byte[1024];
              int i;
              //读取到输入流数据，然后写入到输出流中去，实现复制
              while ((i = inputStream.read(bytes)) != -1) {
                  outputStream.write(bytes, 0, i);
              }
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              try {
                  if (inputStream != null)
                      inputStream.close();
                  if (outputStream != null)
                      outputStream.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  ~~~

  ~~~java
   /**
       * 用filechannel进行文件复制
       *
       * @param fromFile 源文件
       * @param toFile   目标文件
       */
      public void fileCopyWithFileChannel(File fromFile, File toFile) {
          FileInputStream fileInputStream = null;
          FileOutputStream fileOutputStream = null;
          FileChannel fileChannelInput = null;
          FileChannel fileChannelOutput = null;
          try {
              fileInputStream = new FileInputStream(fromFile);
              fileOutputStream = new FileOutputStream(toFile);
              //得到fileInputStream的文件通道
              fileChannelInput = fileInputStream.getChannel();
              //得到fileOutputStream的文件通道
              fileChannelOutput = fileOutputStream.getChannel();
              //将fileChannelInput通道的数据，写入到fileChannelOutput通道
              fileChannelInput.transferTo(0, fileChannelInput.size(), fileChannelOutput);
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (fileInputStream != null)
                      fileInputStream.close();
                  if (fileChannelInput != null)
                      fileChannelInput.close();
                  if (fileOutputStream != null)
                      fileOutputStream.close();
                  if (fileChannelOutput != null)
                      fileChannelOutput.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  ~~~

### ByteArrayInputStream

~~~java
public ByteArrayInputStream(byte buf[]);
public ByteArrayInputStream(byte buf[], int offset, int length) ;
public synchronized int read();
public synchronized int read(byte b[], int off, int len) ;
public synchronized long skip(long n) ;
public synchronized int available();
public boolean markSupported();
public void mark(int readAheadLimit);
public synchronized void reset();
public void close() throws IOException;
~~~

### FilterInputStream

**组合包装过滤器**

- 如：BufferedInputStream buffer = new BuffededInputStream(new FileInputStream(path));

#### BufferedInputStream

- implements FilterInputStream

~~~java
private InputStream getInIfOpen() throws IOException;
private byte[] getBufIfOpen() throws IOException;
public BufferedInputStream(InputStream in);
public BufferedInputStream(InputStream in, int size);
private void fill() throws IOException;
public synchronized int read() throws IOException;
private int read1(byte[] b, int off, int len) throws IOException;
public synchronized int read(byte b[], int off, int len) throws IOException;
public synchronized long skip(long n) throws IOException;
public synchronized int available() throws IOException;
...
~~~

#### DataInputStream

~~~java
public DataInputStream(InputStream in);
public final int read(byte b[]) throws IOException;
public final int read(byte b[], int off, int len) throws IOException;
public final void readFully(byte b[]) throws IOException;
public final void readFully(byte b[], int off, int len) throws IOException ;
public final int skipBytes(int n) throws IOException;
public final boolean readBoolean() throws IOException;
public final byte readByte() throws IOException;
public final int readUnsignedByte() throws IOException;
public final short readShort() throws IOException;
public final int readUnsignedShort() throws IOException;
public final char readChar() throws IOException;
public final int readInt() throws IOException;
public final long readLong() throws IOException ;
public final float readFloat() throws IOException;
public final double readDouble() throws IOException;
public final String readLine() throws IOException;
public final String readUTF() throws IOException;
public final static String readUTF(DataInput in) throws IOException;

~~~

