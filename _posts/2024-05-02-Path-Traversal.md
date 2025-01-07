---
title: "#01: Path traversal"
data: 22024-05-02 00:00:00 +0000
categories: [Web, secure code]
tags: [secure code, web]
---
Path traversal (also known as directory traversal) vulnerabilities enable an attacker to interact with arbitrary files on the server, giving them access to sensitive data. If they can also write to these files, they can potentially modify application data or behavior, ultimately taking full control of the server. <br>

### Tool kit: [DotDotPwn](https://github.com/wireghoul/dotdotpwn), nodeJS, kali linux
![Portswigger](https://miro.medium.com/v2/resize:fit:720/format:webp/1*KVBodG6T0iv642HxZkf9Rg.jpeg)

[File path traversal, simple case (relative path):](https://portswigger.net/web-security/file-path-traversal/lab-simple) <br>
Here the directory /vulnerable takes parameter file and concatenate this file name to the path ```/path/to/directory/``` without any validations.

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
app.get('/vuln', (req, res) => {
  const filename = req.query.file;
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }
// look here
  const filePath = path.join(__dirname, filename);
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      res.status(500).send('Error reading file');
    } else {
      res.send(data);
    }
  });
});
app.get('/', (req, res) => {
  res.send('Hi to 1st Lab Path Traversal, the /vuln endpoint is what you want');
});
app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

[File path traversal, traversal sequences blocked with (absolute path bypass):](https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass)
<br>
The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory. <br>
Many applications that place user input into file paths implement defenses against path traversal attacks. These can often be bypassed.<br>
If an application strips or blocks directory traversal sequences from the user-supplied filename, it might be possible to bypass the defense using a variety of techniques. <br>

You might be able to use an absolute path from the filesystem root, such as ```filename=/etc/passwd```, to directly reference a file without using any traversal sequences.

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
app.get('/vuln', (req, res) => {
  let filePath = req.query.file;
  
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }
  if (filePath.includes('../')) {
    res.send('Hacker caught!');
    return;
  }
// look here
  filePath = path.resolve(__dirname, filePath);
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      res.send('Error reading file.');
    } else {
      res.send(data);
    }
  });
});
app.get('/', (req, res) => {
  res.send('Hi to 2nd Lab Path Traversal, the /vuln endpoint is what you want');
});
app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```
[File path traversal, traversal sequences stripped non-recursively:](https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively)
<br>

You might be able to use nested traversal sequences, such as ```....//``` or ```....\\/```. These revert to simple traversal sequences when the inner sequence is stripped.<br>
“traversal sequences stripped non-recursively” means that the server is attempting to mitigate path traversal attacks by removing traversal sequences from the provided filename. However, the term “non-recursively” indicates that this mitigation strategy is only applied once, without recursively repeating the process. <br>
Essentially, the server is attempting to sanitize the filename by removing any occurrences of traversal sequences, such as ```../```, from the input. However, it does this only once and does not continue to strip traversal sequences that may be embedded within the resulting string after the initial sanitization.<br>

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
app.get('/vuln', (req, res) => {
  let filename = req.query.file;
  
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }
  // Mitigate path traversal by removing traversal sequences once
  filename = filename.replace(/\\.\\.\\//g, '');
  const filePath = path.join(__dirname, filename);
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      if (err.code === 'ENOENT') {
        res.status(404).send('File not found');
      } else {
        res.status(500).send('Internal server error');
      }
    } else {
      res.send(`${data}`);
    }
  });
});
app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```
[File path traversal, traversal sequences stripped with superfluous URL-decode:](https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode)
<br>

In some contexts, such as in a URL path or the filename parameter of a ```multipart/form-data``` request, web servers may strip any directory traversal sequences before passing your input to the application. You can sometimes bypass this kind of sanitization by URL encoding, or even double URL encoding, the ```../``` characters. This results in ```%2e%2e%2f``` and ```%252e%252e%252f``` respectively. Various non-standard encodings, such as ```..%c0%af``` or ```..%ef%bc%8f```, may also work.

[File path traversal, validation of start of path:](https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path)
<br>

An application may require the user-supplied filename to start with the expected base folder, such as ```/var/www/images```. In this case, it might be possible to include the required base folder followed by suitable traversal sequences. For example: ```filename=/var/www/images/../../../etc/passwd```

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
// Define the base folder
// You can change it to your base folder
const baseFolder = '/home/kali/VulnerableLabs/PathTraversal/lab5';
// Vulnerable endpoint
app.get('/vuln', (req, res) => {
    // Get the filename parameter from the query string
    const filename = req.query.filename;
    
    if (!filename) {
        return res.status(400).send('Provide filename parameter \\n\\n your working directory is /home/kali/VulnerableLabs/PathTraversal/lab5');
    }
    // Check if the filename starts with the expected base folder
    if (!filename.startsWith(baseFolder)) {
        return res.status(400).send('Invalid filename');
    }
    // Construct the full path to the file
    const filePath = path.normalize(filename);
    // Read the file content
    fs.readFile(filePath, 'utf8', (err, data) => {
        if (err) {
            return res.status(500).send('Error reading file');
        }
        // Display the file content in the response
        res.send(`${data}`);
    });
});
app.get('/', (req, res) => {
  res.send('Hi to 5th Lab Path Traversal, the /vuln endpoint is what you want');
});
// Start the server
app.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```
<details>
  <summary>In this code:</summary>
  - We define the base folder as /var/www/images.<br>
  - The /vuln endpoint takes a query parameter filename, which is expected to be a file path relative to the base folder.<br>
  - We check if the provided filename starts with the base folder to prevent directory traversal attacks.<br>
  - If the filename is valid, we construct the full path to the file using path.normalize() to ensure it's a valid path. <br>
  - Then, we read the content of the file using fs.readFile() and send it as a response to the client, along with a message indicating the vulnerability and the endpoint to exploit it.
</details>

[File path traversal, validation of file extension with null byte bypass:](https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass) <br>
An application may require the user-supplied filename to end with an expected file extension, such as .png. In this case, it might be possible to use a null byte to effectively terminate the file path before the required extension. For example: ```filename=../../../etc/passwd%00.png```.<br>
Let’s discuss why the null byte (%00) mechanism can bypass the restriction of ending the file name with ```.png```:<br>
When a web application validates user input for a filename to end with a specific extension like .png, it usually checks for the presence of that extension at the end of the filename string. However, this validation is often performed by using string comparison or regular expressions which might not consider null bytes as valid characters. <br>
When ```%00``` (null byte) is appended to the filename, it effectively terminates the string at that point. So, if the filename is ```../../../etc/passwd%00.png```, after the null byte, everything following it, including the .png extension, is ignored by the file system. Therefore, the actual file being accessed becomes ```../../../etc/passwd```, effectively bypassing the restriction of ending the filename with ```.png```. <br>
This bypass occurs because the application’s validation logic doesn’t recognize the null byte as a valid character and stops processing the filename at that point, allowing an attacker to traverse directories and access sensitive files like ```/etc/passwd```.

[How to prevent a path traversal attack](https://portswigger.net/web-security/file-path-traversal) <br>
The most effective way to prevent path traversal vulnerabilities is to avoid passing user-supplied input to filesystem APIs altogether. Many application functions that do this can be rewritten to deliver the same behavior in a safer way. <br>
If you can’t avoid passing user-supplied input to filesystem APIs, we recommend using two layers of defense to prevent attacks: <br>
- Validate the user input before processing it. Ideally, compare the user input with a whitelist of permitted values. If that isn’t possible, verify that the input contains only permitted content, such as alphanumeric characters only.
- After validating the supplied input, append the input to the base directory and use a platform filesystem API to canonicalize the path. Verify that the canonicalized -I described this below-path starts with the expected base directory.
  
Here’s a basic Node.js code example demonstrating how to prevent path traversal attacks using the recommended approach:

```javascript
const path = require('path');
const fs = require('fs');
// Function to validate user input
function validateInput(userInput) {
    // Implement your validation logic here
    // For example, allow only alphanumeric characters
    const alphanumericRegex = /^[a-zA-Z0-9]+$/;
    return alphanumericRegex.test(userInput);
}
// Function to process user input and prevent path traversal
function processInput(baseDir, userInput) {
    // Validate user input
    if (!validateInput(userInput)) {
        throw new Error('Invalid input');
    }
    // Append user input to base directory
    const userInputSafe = path.normalize(userInput);
    const fullPath = path.join(baseDir, userInputSafe);
    // Canonicalize the path
    const canonicalPath = path.resolve(fullPath);
    // Verify that the canonical path starts with the expected base directory
    if (!canonicalPath.startsWith(path.resolve(baseDir))) {
        throw new Error('Invalid path');
    }
    // Proceed with filesystem operation using the safe path
    // For example, read or write files
    fs.readFile(canonicalPath, (err, data) => {
        if (err) {
            console.error('Error reading file:', err);
        } else {
            console.log('File content:', data.toString());
        }
    });
}
// Example usage
const baseDirectory = '/path/to/base/directory';
const userInput = '../file.txt'; // User-supplied input
processInput(baseDirectory, userInput);
```
<details>
  <summary>In this code:</summary>
  - validateInput function validates the user input. You can customize this function according to your requirements. <br>
  - processInput function processes the user input by appending it to the base directory, canonicalizing the path, and verifying that the canonical path starts with the expected base directory.<br>
  - If any validation or verification fails, an error is thrown, preventing a potential path traversal attack.<br>
  - Ensure to replace /path/to/base/directory with the actual base directory in your application. <br>
</details>
Remember that this is a basic example. Depending on specific use case and requirements, you may need to enhance the validation and error handling logic. Additionally, consider implementing additional security measures such as access controls and permissions to further mitigate risks. <br>

### Note:
Canonicalizing a path means converting it to its standardized, absolute form. This process resolves any symbolic links, redundant separators, and references to parent directories (like ..) to produce a unique and normalized representation of the path.<br>
For example, consider the following paths:
- /var/www/project/../index.html
- /var/www/index.html
- /usr/local/bin/../bin/ls

Canonicalizing these paths would result in:
- /var/www/index.html
- /var/www/index.html
- /usr/local/bin/ls
  
As you can see, canonicalization simplifies the path to its most direct form without any unnecessary elements like .. or redundant separators. This ensures consistency and helps prevent ambiguity or confusion when dealing with filesystem paths. <br>
In the context of preventing path traversal attacks, canonicalization helps ensure that user-supplied paths cannot escape from the intended directory structure. By resolving all components of the path to their absolute form, you can accurately determine whether the path stays within the boundaries of the expected directory hierarchy. <br>

### Helpful comment
![screen1](https://miro.medium.com/v2/resize:fit:640/format:webp/1*AGj11vWCGU6XANwp4AtNWw.png)

![screen2](https://miro.medium.com/v2/resize:fit:640/format:webp/1*1ofO8u4x21o1uyYO9UP7aQ.png)