+++
title = "Multiple ways of reading a File in Java"
description = "Multiple ways of reading a File in Java"
tags = [
    "Java",
]
date = "2017-11-18T15:05:00+05:30"
categories = [
    "Java",
]
thumbnail = "img/files.jpg"
+++


### Reading a file line by line from File system path using Reader as character stream

{{< highlight java >}}
public void usingBufferReader() throws IOException 
{
    BufferedReader br = null;
    try {
        File file = new File("D:/Neha/Notes_On_Collections_Java.txt");
        FileReader fr = new FileReader(file);
        br = new BufferedReader(fr);
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        br.close();
    }
}
{{</ highlight >}}

### Reading a file line by line from class path using Reader as character stream
{{< highlight java >}}
public void readFromClassPath() throws IOException 
{
    BufferedReader br = null;
    try 
    {
        // Get file from resources folder
        ClassLoader classLoader = getClass().getClassLoader();
        File file = new File(classLoader.
        			getResource("Notes_On_Collections_Java.txt").getFile());
        br = new BufferedReader(new FileReader(file));
        String line = "";
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        br.close();
    }
}
{{< / highlight >}}

### Reading a file character by character from File system path using InputStream
{{< highlight java >}}
public void usingFileInputStream() 
{
    File file = new File("D:/Neha/Notes_On_Collections_Java.txt");
    FileInputStream fis = null;

    try {
        fis = new FileInputStream(file);
        System.out.println("Total file size to read (in bytes) : " 
        		+ fis.available());

        int content;
        while ((content = fis.read()) != -1) {
            // convert to char and display it
            System.out.print((char) content);
        }

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
{{< / highlight >}}

### Reading a file line by line from class path using BufferedReader
{{< highlight java >}}
public void usingFileInputStream_ClassPath() 
{
    BufferedReader reader = null;
    InputStream in = this.getClass().
    				getResourceAsStream("/Notes_On_Collections_Java.txt");
    try {

        reader = new BufferedReader(new InputStreamReader(in));
        String content;
        while ((content = reader.readLine()) != null) {
            System.out.println(content);
        }

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
{{< / highlight >}}

### Reading a file line by line from class path using Java8 Files, Stream
{{< highlight java >}}
public void readFileAsStream() 
{
    ClassLoader classLoader = getClass().getClassLoader();
    File file = new File(classLoader.
    			getResource("Notes_On_Collections_Java.txt").getFile());
    Stream<String> stream=null;
    try {
        stream = Files.lines(Paths.get(file.getPath()));
        stream.forEach(System.out::println);
    } catch (IOException e) {
        e.printStackTrace();
    }
    finally {
        stream.close();
    }

}
{{< / highlight >}}

### Reading a file line by line from class path using Commons-IO FileUtils
{{< highlight java >}}
public void usingFileIOUtils() 
{
    try {
        ClassLoader classLoader = getClass().getClassLoader();
        File file = new File(classLoader.
        			getResource("Notes_On_Collections_Java.txt").getFile());
        List<String> lines = FileUtils.readLines(file, "UTF-8");
        for (String string : lines) {
            System.out.println(string);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
{{< / highlight >}}
