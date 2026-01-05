### mpy-mbedtls

MicroPython bindings for ECDSA keys basic functionality and x509 cert/csr utilities.
*Supports both PEM and DER formats*

#### Features:

`mbedtls` module (low level):

   - Generate ECDSA key pair
	
   - Derive public key from private key
	
   - Sign data
    
   - Verify signature

`x509` module:
	
   - Generate a certificate signing request (CSR)
	
   - Parse certificate
	
   - Verify certificate
   
`ecdsa` module (Same as mbedtls but OOP):
   
   - Generate ECDSA key pair
	
   - Derive public key from private key
   
   - Parse private/public key file
	
   - Sign data
    
   - Verify signature
   
   - Sign file 
   
   - Verify file signature
   
   - Export private/public key to file


# Building for ESP32 Generic

## Pre Requisites

## Ensure `esp-idf` is installed within your environment. 

To accomplish this, do the following:

1. 

```console
git clone https://github.com/espressif/esp-idf.git
```

2. Change directories to the repo: `cd esp-idf`
3. Change version to `v5.5.1`:
```console
git checkout v5.5.1
```
5. Download submodules:
```console
git submodule update --init --recursive
```
7. Install using the `install.sh` script: `./install.sh`
8. Once it's finalized, you can start the environment that runs esp-idf with: `. ./export.sh`

## Building the Firmware with mbedtls

1. Clone the micropython repository:

```console
git clone https://github.com/micropython/micropython.git
```

2. Change directories to the repo: `cd micropython`
3. Ensure submodules have been updated:
```console
git submodule update --init --recursive
```
4. Move to the `mpy-cross` folder within micropython and re-build:

```console
make clean
make -j4
```
5. Within your home directory, clone the mbedtls repository
```console
git clone https://github.com/Carglglz/mpy-mbedtls.git
```
6. Move back into the micropyton repo, specifically the `ports/esp32` folder and make a folder for mbedtls: `mkdir mbedtls`
7. Build the firmware (ensure `cmake` is installed first `sudo apt install cmake -y`):
```console
make BOARD=ESP32_GENERIC USER_C_MODULES=/absolute/path/to/mpy-mbedtls/mbedtls/micropython.cmake FROZEN_MANIFEST=/absolute/path/to/mpy-mbedtls/ports/esp32/mainfest.py -j4
```

## Flashing the Firmware

Once the firmware has been built, it'll be located within `micropython/ports/esp32/build-ESP32_GENERIC/`, the `firmware.bin` file is what you'll need.

1. Ensure you have esptool installed on your machine
2. Plug your ESP32 device into your machine and wipe it:
```console
esptool --port /dev/ttyUSB0 erase-flash
```
Note: Ensure you're targeting the proper port here
3. Flash the firmware file to the board:
```console
esptool --port /dev/ttyUSB0 --baud 460800 write-flash 0x1000 /path/to/micropython/ports/esp32/build-ESP32_GENERIC/firmware.bin
```
4. That's it! You should now be able to plug your ESP32 device in and use something like Thonny or Arduino Lab for Micropython to access the console. If you can successfully run: `import mbedtls` without any errors, it's successfully been flashed!

## Examples

Below is an example of how a message can be signed using mbedtls.

```python
import mbedtls

curve = 'secp256r1'

priv_key, pub_key = mbedtls.ec_gen_key(curve)

message = b"Test message to sign"

signature = mbedtls.ec_key_sign(priv_key, message)

# Verify signature, returns True if valid
result = mbedtls.ec_key_verify(pub_key, message, signature)

print("Signature valid? ", result)
```

# Building for Unix

1. Ensure you have the micropython and mbedtls repositories cloned to your machine: 

```console
git clone https://github.com/micropython/micropython.git
git clone https://github.com/Carglglz/mpy-mbedtls.git
```
2. Navigate to the `extmod` folder and cd into the `mbedtls` folder in the micropython repo.
3. Add the following definitions in the `mbedtls_config_common.h` file:

```console
#define MBEDTLS_PK_WRITE_C
#define MBEDTLS_X509_CREATE_C
#define MBEDTLS_X509_CSR_WRITE_C
#define MBEDTLS_PEM_WRITE_C
#define MBEDTLS_BASE64_C
#define MBEDTLS_OID_C
#define MBEDTLS_ASN1_WRITE_C
```

4. Save the file and navigate to the `ports/unix` folder within the micropython repository.
5. Build micropython:
```console
make USER_C_MODULES=/path/to/mbedtls/repository/mpy-mbedtls FROZEN_MANIFEST=/path/to/mbedtls/repository/mpy-mbedtls/ports/unix/manifest.py -j$(nproc)
```
*Note*: If this fails, you may need to navigate to the `mpy-cross` folder within the micropython repo and run: `make clean && make`. Additionally, ensure you have the following packages installed: `build-essential python3 libffi-dev git pkg-config`. If not, you can install with `sudo apt install build-essential python3 libffi-dev git pkg-config -y`.

6. Navigate to the new folder this creates and execute micropython with `./micropython`
7. You should then be able to run `import mbedtls` with no errors

#### Run tests

In `micropython/tests`

```
$ ./run-tests.py ../../<path to user modules>/mpy-mbedtls/tests/test_*.py
pass  ../../user_modules/mpy-mbedtls/tests/test_mbedtls_ec_curves.py
pass  ../../user_modules/mpy-mbedtls/tests/test_mbedtls_ec_keyp_der.py
pass  ../../user_modules/mpy-mbedtls/tests/test_mbedtls_ec_keyp.py
pass  ../../user_modules/mpy-mbedtls/tests/test_x509_cert_parse.py
pass  ../../user_modules/mpy-mbedtls/tests/test_ecdsa.py
pass  ../../user_modules/mpy-mbedtls/tests/test_x509_gen_csr.py
pass  ../../user_modules/mpy-mbedtls/tests/test_x509_cert_validate.py
7 tests performed (19 individual testcases)
7 tests passed

```

#### Example 

```python
import ecdsa

keyp = ecdsa.ECKeyp()

print("PRIVATE KEY:")
print(keyp.pkey.decode())

print("PUBLIC KEY:")
print(keyp.pubkey.decode())

msg = "hello world"

# Sign
signature = keyp.sign(msg)

assert isinstance(signature, bytes)
print("Signature: OK")

# Verify signature

verification = keyp.verify(msg, signature)

assert verification
print("Verification: OK")


```

```
>>> import example
PRIVATE KEY:
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIGrK/lMb3BvPEk2LhBmRWj01duluiI/qagOFQaXfGIOAoAoGCCqGSM49
AwEHoUQDQgAEzKw4gnXWWVfNy2dP6WYzJ4UN/E5DPhyJdUEtYC4j8PvXTnFPdpga
XXN+n0oofGF/aTfwX3UqNkc+qvUKtkPzKg==
-----END EC PRIVATE KEY-----

PUBLIC KEY:
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEzKw4gnXWWVfNy2dP6WYzJ4UN/E5D
PhyJdUEtYC4j8PvXTnFPdpgaXXN+n0oofGF/aTfwX3UqNkc+qvUKtkPzKg==
-----END PUBLIC KEY-----

Signature: OK
Verification: OK
```


See other examples in `mpy-mbedtls/tests`
