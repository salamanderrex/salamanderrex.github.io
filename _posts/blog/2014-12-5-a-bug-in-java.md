---
layout: post
title: A bug in JAVA
description: a bug in the java IO. It happens when you want to delete a file immediately after you write something to it.
category: blog
---

##A bug in JAVA

Recently I found a bug in JAVA. It rarely happens, but as a development tool, this is unavoidable.
This bug occurs when you want to delete a file after you write something to it or create it.


####Example
First, you write to a file, like get a input stream from other file and then close the IOstream.

		File file=new File(serverLocation);
		OutputStream  outpuStream = new FileOutputStream(file);;
		try {
			int read = 0;
			byte[] bytes = new byte[1024];

			outpuStream = new FileOutputStream(new File(serverLocation));
			while ((read = uploadedInputStream.read(bytes)) != -1) {
				outpuStream.write(bytes, 0, read);
			}
			outpuStream.flush();
			outpuStream.close();
			uploadedInputStream.close();
			
After some steps, you may want to delete it.

		public boolean deleteFile(String sPath) throws Exception {
		boolean flag = false;
		File  file = new File(sPath);
        if (file.isFile() && file.exists()) {
            file.delete();
            flag = true;
        }
        return flag;
    }
	
Then the weird thing happens. When you use the deleteFile, it will return `Ture` but the file is still on the disk.
In addition, in JAVA, this is no such thing like `file.close()`, and only IO stream can be closed. File is just a description here.


##How to solve this question
####method one (Not recommended)
try to cause the Exception and it will close the file stream.

	try{
		outpuStream.close();
		outpuStream.write(bytes,0,1);
	}
	catch (Exception ignore)
	{
	}
	
####method two
Very Simple, just use `System.gc()` .
After you write something,stream can be closed in the `finally` part.


	finally{
		try
		{
			in.close();
			in = null;
			out.flush();
			out.close();
			out = null;
			System.gc();
		}
		catch (IOException e)
		{
			logger.error(e.getMessage());
			e.printStackTrace();
		}
	}
	
##Is is bug??
Unfortunately, this is a bug of JAVA and this cannot be fixed now. You can check this link:

[Bug report @ java.com 2002-07-16][Javabug]


Here is the final comment:

>We cannot fix this.  Windows does not allow a mapped file to be deleted.  This
problem should be ameliorated somewhat once we fix our garbage collectors to
deallocate direct buffers more promptly (see 4469299), but otherwise there's
nothing we can do about this.

This is bug reported at 2002. It is 2014 now but the bug is still not fixed. It seems that JAVA still needs a long way to develop itself.

[Javabug]: http://bugs.java.com/bugdatabase/view_bug.do?bug_id=4715154
			