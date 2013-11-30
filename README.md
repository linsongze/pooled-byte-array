pooled-byte-array
=================

byte[] pool - Automatically recycled by Garbage Collector


Assumptions (for relatively big byte[])
--------------
- byte[] are expensive to allocate (zeroed)
- consume young gen unnecessary => increase frequency of minor GC
- if promoted => increase minor GC pause because of copy between gens

byte[] can be reused manually but in algorithms that don't control the full reference or life cycle of the byte[], it's nice to let the garbage collector do his job and find out when the byte[] is not used anymore.
Reusing byte[] manually, sometimes means thread-safety/lock issues or over-provisionning.

This experimental approach try to cover these problems.



Usage
-----

```java
// Instanciate the factory once
// (Here with array of 1024 bytes and 100 of them pre-allocated)
PooledByteArrayFactory factory = new PooledByteArrayFactory(1024, 100);
```

```java
// Anywhere in the code
// Instead of 
// byte[] buffer = new buffer[1024];
// you call the factory for a new byte buffer
PooledByteArray buffer = factory.getByteArray();
```

**You don't need to manualy recycle the PoolByteArray object back into the pool.
The wrapped byte[] in PooledByteArray will be recycled in the factory automatically
when the Garbage Colector will reclaim the PooledByteArray.**

You can then use the PooledByteArray for all sort of operation:

```java
public class PooledByteArray
{
    ...
    public int length()
    public byte getAt(int index)
    public void setAt(int index, byte b)
    public void copyTo(byte[] dst, int dstPos, int offset, int len)
    public void copyFrom(byte[] src, int srcPos, int offset, int len)
    public void writeTo(OutputStream out, int offset, int len) throws IOException
    public int readFrom(InputStream in, int offset, int len) throws IOException
    ...
}
```
To be enriched...



Careful Notes
-------------

* PooledByteArray are not zeroed like new allocated byte[]

* Obviously you need to be careful with references to PooledByteArray, but it's not different than a reference to the byte[].

* The garbage collector needs to kick-in in order to recycle byte arrays into the pool.
So if the PooledByteArray survives the young generation, then only a FullGC or a ConcMarkSweep of the old gen is going to recyle the wrapped byte[]. I guess it works great with Azul Zing GC.



