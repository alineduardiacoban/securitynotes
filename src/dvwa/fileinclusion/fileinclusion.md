# File Inclusion

## Security Level - Low

![file-inclusion-low](images/file-inclusion-low.png)

## Source Code

```php
 <?php

// The page we wish to display
$file = $_GET[ 'page' ];

?>
```
If we choose one of the files: 

![requestfileinclusionlow](images/requestfileinclusionlow.png)

### Local File Inclusion

**Request**:
![local-file-inclusion-low-level-request](images/local-file-inclusion-low-level-request.png)

**Browser**:
![browser-local-file-inclusion-low-level-request](images/browser-local-file-inclusion-low-level-request.png)

### Remote File Inclusion

![remote-file-inclusion](images/remote-file-inclusion.png)

