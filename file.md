## File

### File upload
``` ts
"use client"
import { useState } from 'react';

interface FileUploadProps {
  userId: number;
}

const FileUpload = ({ userId }: FileUploadProps) => {
  const [file, setFile] = useState<File | null>(null);
  const [message, setMessage] = useState<string | null>(null);

  const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    if (event.target.files) {
      setFile(event.target.files[0]);
    }
  };

  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();
    if (!file) return;

    const formData = new FormData();
    formData.append('file', file);
    formData.append('userId', userId.toString());

    try {
      const response = await fetch('http://localhost:8080/api/files/upload', {
        method: 'POST',
        body: formData,
      });

      if (response.ok) {
        setMessage('File uploaded successfully');
        setFile(null); // Reset file input
      } else {
        setMessage('Error uploading file');
      }
    } catch (error) {
      setMessage('Error uploading file');
    }
  };

  return (
    <div>
      <h2>Upload a File</h2>
      <form onSubmit={handleSubmit}>
        <input type="file" onChange={handleFileChange} />
        <button type="submit">Upload</button>
      </form>
      {message && <p>{message}</p>}
    </div>
  );
};

export default FileUpload;

```


### File List
```ts
"use client"
import { useState, useEffect } from 'react';

interface FileMetaData {
  id: number;
  fileName: string;
  filePath: string;
}

interface FileListProps {
  userId: number;
}

const FileList = ({ userId }: FileListProps) => {
  const [files, setFiles] = useState<FileMetaData[]>([]);

  useEffect(() => {
    const fetchFiles = async () => {
      try {
        const response = await fetch(
          `http://localhost:8080/api/files/user/${userId}`
        );

        if (response.ok) {
          const data = await response.json();
          console.log(data);
          setFiles(data);
        } else {
          console.error('Failed to fetch files');
        }
      } catch (error) {
        console.error('Error fetching files', error);
      }
    };

    fetchFiles();
  }, [userId]);

  return (
    <div>
      <h2>Uploaded Files</h2>
      <ul>
        {files.map((file) => (
          <li key={file.id}>
            <a
              href={`http://localhost:8080/api/files/download/${file.fileName}`}
              target="_blank"
              rel="noopener noreferrer"
            >
              {file.fileName}
            </a>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default FileList;

```
