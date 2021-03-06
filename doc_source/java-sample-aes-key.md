# Generate an AES Symmetric Key<a name="java-sample-aes-key"></a>


|  | 
| --- |
|  \*\* Example code only \- Not for production use \*\* This page includes example code that has not been fully tested\. It is designed for test environments\. Do not run this code in production\.  | 

This example shows how to generate a 256\-bit Advanced Encryption Standard \(AES\) symmetric key and save it in an HSM\. By default, the keys that the HSM generates are not saved\. To save a key, call the `makeKeyPersistant` method below\. You can save the key object and use the key handle in other operations\. 

**Note**  
This example uses the `loginWithEnvVariables()` method in the [Log In To and Out Of an HSM](java-sample-login.md) sample to log in to the HSM\. You can substitute the login method that you prefer\. Also, the example assumes that [the Cavium provider](use-cavium-provider.md) is included in your Java provider file\. If it is not, create an instance of the provider and substitute it for the **Cavium** string\. 

```
package com.amazonaws.cloudhsm.examples.key.symmetric;

import java.security.Key;
import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;

import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;

import com.amazonaws.cloudhsm.examples.operations.LoginLogoutExample;
import com.cavium.cfm2.CFM2Exception;
import com.cavium.cfm2.Util;
import com.cavium.key.CaviumAESKey;
import com.cavium.key.parameter.CaviumAESKeyGenParameterSpec;
import com.cavium.provider.CaviumProvider;

public class AESSymmetricKeyGeneration {

  public static void main(String[] z) {
    LoginLogoutExample.loginWithEnvVariables();
    new AESSymmetricKeyGeneration().generateAESKey(256, true);
    new AESSymmetricKeyGeneration().generateAESKey(256, "AESKeyCustomLabel", false, true);
    LoginLogoutExample.logout();
  }

  // Generate a key given the bit size and a Boolean value that specifies whether the key is
  // to be added to the HSM (persistent)
  public Key generateAESKey(int keySize, boolean isPersistent) {
    KeyGenerator keyGen;
    
    try {

      // Create an instance of the provider.
      keyGen = KeyGenerator.getInstance("AES","Cavium");
      
      // Generate the key.
      keyGen.init(keySize);
      SecretKey aesKey = keyGen.generateKey();
      System.out.println("Key Generated!");
      
      if(aesKey instanceof CaviumAESKey) {
        System.out.println("Key is of type CaviumAESKey");
        CaviumAESKey caviumAESKey = (CaviumAESKey) aesKey;
        
        // Save the key handle. You'll need this to perform future encryption and decryption operations.
        System.out.println("Key Handle = " + caviumAESKey.getHandle());
        
        // Get the key label generated by the SDK.
        System.out.println("Key Label = " + caviumAESKey.getLabel());
        
        // Get the Extractable property of the key.
        System.out.println("Is Key Extractable? : " + caviumAESKey.isExtractable());
        
        // Get the Persistent property of the key.
        System.out.println("Is Key Persistent? : " + caviumAESKey.isPersistent());
        
        // By default, keys are not persistent. Make them Persistent here.
        if(isPersistent){
          System.out.println("Setting Key as Persistent:");
          makeKeyPersistant(caviumAESKey);
          System.out.println("Key is Persistent!");
        }
        System.out.println("Is Key Persistent? : " + caviumAESKey.isPersistent());

        // Verify the key type and size.
        System.out.println("Key Algo : " + caviumAESKey.getAlgorithm());
        System.out.println("Key Size : " + caviumAESKey.getSize());
      }
      
      return aesKey;
      
    } catch (NoSuchAlgorithmException e) {
      e.printStackTrace();
    } catch (NoSuchProviderException e) {
      e.printStackTrace();
    } catch (Exception e) {
      e.printStackTrace();
    } 
    return null;
  }

  // This method allows you to specify the size in bits of the key, the private key label, whether the private key 
  // can be extracted, and whether the key is persistent.
  public Key generateAESKey(int keySize,String keyLabel, boolean isExtractable, boolean isPersistent) {
    KeyGenerator keyGen;
    
    try {
      // Create an instance of the provider.
      keyGen = KeyGenerator.getInstance("AES","Cavium");
      
      // Generate the key.
      CaviumAESKeyGenParameterSpec aesSpec = new CaviumAESKeyGenParameterSpec(keySize, keyLabel, isExtractable, isPersistent);
      keyGen.init(aesSpec);
      SecretKey aesKey = keyGen.generateKey();
      System.out.println("Key Generated!");
      
      if(aesKey instanceof CaviumAESKey) {
        System.out.println("Key is of type CaviumAESKey");
        CaviumAESKey caviumAESKey = (CaviumAESKey) aesKey;
        
        // Save the key handle. You'll need this to perform future encryption and decryption operations.
        System.out.println("Key Handle = " + caviumAESKey.getHandle());
        
        // Get the key label. 
        System.out.println("Key Label = " + caviumAESKey.getLabel());
        
        // Get the Extractable property of the key.
        System.out.println("Is Key Extractable? : " + caviumAESKey.isExtractable());
        
        //Get the Persistent property of the key.
        System.out.println("Is Key Persistent? : " + caviumAESKey.isPersistent());

        // Verify the key type and size.
        System.out.println("Key Algo : " + caviumAESKey.getAlgorithm());
        System.out.println("Key Size : " + caviumAESKey.getSize());
      }
      
      return aesKey;
      
    } catch (NoSuchAlgorithmException e) {
      e.printStackTrace();
    } catch (NoSuchProviderException e) {
      e.printStackTrace();
    } catch (Exception e) {
      e.printStackTrace();
    } 
    return null;
  }

  // Save the key to the HSM.
  public static void makeKeyPersistant(Key key) {
    CaviumAESKey caviumAESKey = (CaviumAESKey) key;
    try {
      Util.persistKey(caviumAESKey);
      System.out.println("Added Key to HSM");
    } 
    catch (CFM2Exception e) {
      e.printStackTrace();
    }
  }
```