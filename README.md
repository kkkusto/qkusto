import os
import sys
import hashlib
import base64
import getpass

def derive_key(password: str, salt: bytes, length: int = 32) -> bytes:
    """Derive a key from password using PBKDF2-HMAC-SHA256."""
    return hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000, length)

def xor_decrypt(data: bytes, key: bytes) -> bytes:
    """Decrypt data using XOR with a repeating key (same as encryption)."""
    return bytes(a ^ b for a, b in zip(data, (key * (len(data) // len(key) + 1))[:len(data)]))

def decrypt_file(input_file: str, output_file: str, secret_key: str):
    """Decrypt a file using the provided secret key."""
    try:
        # Read the encrypted file
        with open(input_file, 'rb') as f:
            encoded_data = f.read()
        
        # Decode from base64
        try:
            combined_data = base64.b64decode(encoded_data)
        except Exception:
            raise ValueError("Invalid file format - not a valid encrypted file")
        
        # Check minimum file size (16 bytes salt + at least 1 byte data)
        if len(combined_data) < 17:
            raise ValueError("File too small - not a valid encrypted file")
        
        # Extract salt (first 16 bytes) and encrypted data
        salt = combined_data[:16]
        encrypted_data = combined_data[16:]
        
        # Derive key from password using the same salt
        key = derive_key(secret_key, salt, 32)
        
        # Decrypt the data using XOR
        decrypted_bytes = xor_decrypt(encrypted_data, key)
        
        # Convert bytes back to text
        try:
            decrypted_text = decrypted_bytes.decode('utf-8')
        except UnicodeDecodeError:
            raise ValueError("Decryption failed - wrong password or corrupted file")
        
        # Write decrypted data to output file
        with open(output_file, 'w', encoding='utf-8') as f:
            f.write(decrypted_text)
        
        print(f"✅ File decrypted successfully!")
        print(f"🔐 Encrypted file: {input_file}")
        print(f"📁 Decrypted file: {output_file}")
        print(f"🔑 Key derivation: PBKDF2-HMAC-SHA256 (100,000 iterations)")
        
    except FileNotFoundError:
        print(f"❌ Error: File '{input_file}' not found.")
        sys.exit(1)
    except ValueError as e:
        print(f"❌ Error: {str(e)}")
        sys.exit(1)
    except Exception as e:
        print(f"❌ Error during decryption: {str(e)}")
        sys.exit(1)
