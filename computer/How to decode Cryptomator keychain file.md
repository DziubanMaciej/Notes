parentTopic:: [[Computer]]

# Decode cryptomator keychain file

```powershell
Add-Type -AssemblyName System.Security

function decode {
    param($password, $salt)
    
    $password = [System.Convert]::FromBase64String($password)
    $salt = [System.Convert]::FromBase64String($salt)
    #$scope  = [System.Security.Cryptography.DataProtectionScope]::LocalMachine
    $scope  = [System.Security.Cryptography.DataProtectionScope]::LocalMachine

    $result = [System.Security.Cryptography.ProtectedData]::Unprotect($password, $salt, $scope)
    $result = [System.Text.UTF8Encoding]::UTF8.GetString($result)
    return $result
}
```

How to use:
1. Open `%APPDATA%\Cryptomator\keychain.json`
2. get "ciphertext" and "salt"
3. call `decode <ciphertext> <salt>` in powershell
