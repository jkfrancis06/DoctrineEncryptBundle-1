# Customer encryption class

We can imagine that you want to use your own encryption class, it is simpel.

### Warning: make sure you add the `<ENC>` after your encrypted string.

1. Create an new class and implement Ambta\DoctrineEncryptBundle\Encryptors\EncryptorInterface.
2. Create a constructor with the parameter secret key `__construct($secretKey)`
3. Create a function called encrypt with parameter data `encrypt($data)`
4. Create a function called decrypt with parameter data `decrypt($data)`
5. Insert your own encryption/decryption methods in those functions.
6. Define the class in your configuration file

## Example

### MyRijndael192Encryptor.php

``` php
<?php

namespace YourBundle\Library\Encryptor;

use Ambta\DoctrineEncryptBundle\Encryptors\EncryptorInterface;

/**
 * Class for variable encryption
 * 
 * @author you <you@youremail.com>
 */
class MyRijndael192Encryptor implements EncryptorInterface {

    /**
     * @var string
     */
    private $secretKey;

    /**
     * @var string
     */
    private $initializationVector;

    /**
     * {@inheritdoc}
     */
    public function __construct($key) {
        $this->secretKey = md5($key);
        $this->initializationVector = mcrypt_create_iv(
            mcrypt_get_iv_size(MCRYPT_RIJNDAEL_192, MCRYPT_MODE_ECB),
            MCRYPT_RAND
        );
    }

    /**
     * {@inheritdoc}
     */
    public function encrypt($data) {

        if(is_string($data)) {
            return trim(base64_encode(mcrypt_encrypt(
                MCRYPT_RIJNDAEL_192,
                $this->secretKey,
                $data,
                MCRYPT_MODE_ECB,
                $this->initializationVector
            ))). "<ENC>";
        }

        return $data;

    }

    /**
     * {@inheritdoc}
     */
    public function decrypt($data) {

        if(is_string($data)) {
            return trim(mcrypt_decrypt(
                MCRYPT_RIJNDAEL_192,
                $this->secretKey,
                base64_decode($data),
                MCRYPT_MODE_ECB,
                $this->initializationVector
            ));
        }

        return $data;
    }
}
```

### config.yaml

``` yaml
ambta_doctrine_encrypt:
    secret_key:           AB1CD2EF3GH4IJ5KL6MN7OP8QR9ST0UW # Your own random 256 bit key (32 characters)
    encryptor_class:      \YourBundle\Library\Encryptor\MyRijndael192Encryptor # your own encryption class
```

Now your encryption is used to encrypt and decrypt data in the database.

#### [Back to the index](https://github.com/ambta/DoctrineEncryptBundle/blob/master/Resources/doc/index.md)