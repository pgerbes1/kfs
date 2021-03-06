This tutorial covers everything you need to know about using KFS within your 
application. KFS is based on LevelDB, an embedded key-value store, but the 
interface for interacting with a KFS is focused on the storage and retrieval 
of files and arbitrary binary streams.

### Getting Started

To create and open a new KFS database (or open an existing one), simply 
require the module and create a {@link Btable} object:

```
const kfs = require('kfs');
const myDataStore = kfs('/path/to/database.kfs');
```

That's it, Your data store is ready to use!

### Check if a File Exists

To check if a file exists at a given key, use the {@link Btable#exists} method:

```
const some160bitKey = 'adc83b19e793491b1c6ea0fd8b46cd9f32e592fc';

myDataStore.exists(some160bitKey, (err, exists) => {
  console.log('The file ' + (exists ? 'DOES' : 'DOES NOT') + ' exist!');
});
```

### Check if a File Can Be Stored

To check the available space for a file at a given key, use the 
{@link Btable#getSpaceAvailableForKey} method:

```
const fileSizeInBytes = 4096;

myDataStore.stat(some160bitKey, (err, result) => {
  if (err) {
    // handle error
  }

  let enoughFreeSpace = result.sBucketStats.free > fileSizeInBytes;

  console.log('There ' + (enoughFreeSpace ? 'IS': 'IS NOT') + ' enough space!');
});
```

### Write a File to the Data Store

To write a raw buffer to the data store, use the {@link Btable#writeFile} 
method:

```
const myFileBuffer = Buffer.from([/* ... */]);

myDataStore.writeFile(some160bitKey, myFileBuffer, (err) => {
  console.log('File ' + (err ? 'WAS NOT' : 'WAS') + ' written!');
});
```

### Read a File from the Data Store

To read a file into memory from the data store, use the {@link Btable#readFile} 
method:

```
myDataStore.readFile(some160bitKey, (err, fileBuffer) => {
  console.log(err || fileBuffer);
});
```

### Remove a File from the Data Store

To remove a file from the data store, use the {@link Btable#unlink} method:

```
myDataStore.unlink(some160bitKey, (err) => {
  console.log('The file ' + (err ? 'WAS NOT' : 'WAS') + ' removed!');
});
```

### Use the Streaming Interfaces

When reading or writing larger files, you may not wish to buffer everything 
into memory. In these cases, use the {@link Btable#createReadStream} and 
{@link Btable#createWriteStream} methods:

```
myDataStore.createReadStream(some160bitKey, (err, readableStream) => {
  if (err) {
    // handle error
  }

  readableStream.on('data', (chunk) => {
    console.log('Got chunk:', chunk);
  });

  readableStream.on('end', () => {
    console.log('All chunks read!');
  });

  readableStream.on('error', (err) => {
    console.log('Failed to read file:', err.message);
  });
});
```

```
myDataStore.createWriteStream(some160bitKey, (err, writableStream) => {
  if (err) {
    // handle error
  }

  writableStream.on('finish', () => {
    console.log('All chunks written!');
  });

  writableStream.on('error', (err) => {
    console.log('Failed to write file:', err.message);
  });

  writableStream.write(Buffer.from([/* ... */]));
  writableStream.write(Buffer.from([/* ... */]));
  writableStream.write(Buffer.from([/* ... */]));
  writableStream.end();
});
```
