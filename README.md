# Encrypt-Data-with-AWS-KMS

# Objective 

In this project, I demonstrated how to use encryption to secure data with AWS Key Management Service (KMS). The goal was to create encryption keys in KMS, encrypt a DynamoDB table using that key, and then test access through different IAM users.

# Tools & Concepts Learned

- How AWS KMS secures encryption keys and enforces access through policies.
- Practiced using symmetric encryption and understood why it’s efficient for large data volumes.
- Learned the difference between key administrators (who manage key policies and lifecycle) and key users (who can encrypt/decrypt data).
- Explored DynamoDB’s encryption options: AWS-owned keys, AWS-managed keys, and customer-managed keys.
- Understood transparent data encryption, where authorized users see decrypted data even though it is stored encrypted at rest.
- Gained hands-on experience modifying KMS key policies to enforce fine-grained access control.

# Encryption and KMS
Encryption is the process of converting readable data (plaintext) into an unreadable format (ciphertext) using an algorithm and an encryption key. The goal is to protect sensitive information so that only authorized users with the correct key can turn it back into its original form. Companies and developers use encryption to secure data from unauthorized access, both when itʼs stored (data at rest) and when itʼs being sent over networks (data in transit). AWS Key Management Service (KMS) is a managed service in AWS that creates, stores, and manages encryption keys. It acts like a secure vault for your keys and lets you use them to encrypt and decrypt data across AWS services like S3, EBS, and RDS. Instead of building your own key management system, KMS handles the heavy lifting generating new keys, controlling access with IAM policies, automatically rotating keys if needed, and logging every use in CloudTrail for auditing. I chose a symmetric key because itʼs faster and more efficient for encrypting large amounts of data, like what Iʼm storing in my DynamoDB table. Since the same key is used for both encryption and decryption, the process is quicker and uses fewer resources compared to asymmetric encryption. For my use case, where the data is handled within a trusted environment, symmetric encryption gives me the speed and security I need without the extra overhead.

<img width="685" height="249" alt="Image" src="https://github.com/user-attachments/assets/56f45da4-8ac9-45e3-ac1e-a4522fee7b8e" />

# Encrpyting Data 
 DynamoDB is one of AWS's database services. DynamoDB stands out as a fast and flexible way to store your data, which makes it a great choice for applications that need quick access to large volumes of data e.g. games. DynamoDB offers three different encryption options. The first is Owned by Amazon DynamoDB, where AWS fully manages the encryption key and you donʼt get access or visibility into it. This is best for basic encryption when you donʼt need any control. The second option is an AWS managed key, where AWS KMS manages the key on your behalf. You can see the key and its usage, but AWS still takes care of the management. The third option is a Customer managed key (CMK), which you create, own, and manage in KMS. This gives you full control over the key, including policies, lifecycle, and usage. For this project, I chose the customer managed key since it provides the highest level of security and control.

 <img width="1382" height="390" alt="Image" src="https://github.com/user-attachments/assets/2aff29a3-825e-4d93-a9db-3ffcec876244" />

 # Data Visibility
A KMS key manages user permissions through IAM policies and key policies. These
 policies define who can administer the key and who can use it for cryptographic
 operations like encrypting or decrypting data. For example, a key administrator can
 control the keyʼs lifecycle, attach policies, and decide which IAM users or roles are
 allowed to use the key. A key user, on the other hand, can perform encryption and
 decryption only if the policies grant them permission. This fine-grained control
 ensures that only authorized users can access sensitive data protected by the KMS
 key.
 Even though my DynamoDB table is encrypted, I can still see the items because I have
 permissions to use the encryption key in KMS. DynamoDB automatically decrypts the
 data on my behalf whenever I access it, a process known as transparent data
 encryption. This means the data is always stored securely in an encrypted state at
 rest, but when an authorized user or application retrieves it, DynamoDB uses the key
 to decrypt the data and present it in readable form. As a result, the data stays
 protected while still being instantly accessible to those with the correct permissions

 <img width="1386" height="738" alt="Image" src="https://github.com/user-attachments/assets/352bdd32-6d21-4537-922c-0a48b2b3427c" 

# Denying Access

 I configured a new IAM user to validate whether unauthorized users can access
 encrypted data. The permission policies I granted this user are DynamoDB full access
 but not encryption/decryption permission with AWS KMS.
 When I checked my DynamoDB table as the test user, I saw an Access Denied error.
 The message explained that I did not have permission to use the KMS key for the
 kms:Decrypt action. This confirmed that although the test user had full DynamoDB
 access, they could not view or decrypt the data because they were missing the
 required KMS key permissions

<img width="684" height="328" alt="Image" src="https://github.com/user-attachments/assets/39ae5274-7b9d-4feb-841d-c525ea78877d" 

  # Granting Access
   To let my test user use the encryption key, I updated the KMS key policy to include the
 IAM user as a key user. By modifying the policy, I allowed the test user permissions
 such as Encrypt, Decrypt, ReEncrypt*, GenerateDataKey*, and DescribeKey. This
 change gave the test user the ability to perform cryptographic operations with the key,
 which enabled them to decrypt and access the DynamoDB table data.
 I verified my test userʼs access by logging into AWS with the test user credentials and
 attempting to query items in the DynamoDB table. After updating the KMS key policy
 to grant the test user permissions, I was able to see the decrypted data returned from
 the table. This confirmed that the test user could now use the KMS key to decrypt and
 access the data, proving that the permissions update was successful.
 I would use access control tools like IAM policies, roles, or security groups when I
 need to decide who can access a resource or service, such as a DynamoDB table or
 an S3 bucket. These tools restrict access to the resource itself but donʼt protect the
 actual data once someone has access. I would use encryption when I want to protect
 the data inside those resources. Encryption ensures that even if someone bypasses
 access controls, intercepts traffic, or gains unauthorized access, they still canʼt read
 the data without the proper decryption key. In practice, both are used together: access
 controls decide who can get to the resource, and encryption ensures that the data
 remains secure at rest and in transit

<img width="669" height="461" alt="Image" src="https://github.com/user-attachments/assets/cd10da8b-d5ca-48c2-8647-414130f4781e" />

# Summary 
This project gave me practical experience applying encryption with AWS KMS to secure a DynamoDB table. I built and tested encryption keys, managed IAM permissions, and validated how policies enforce who can and cannot access sensitive data. The key takeaway is that access control and encryption are most effective when combined: IAM policies define who can reach a resource, while KMS ensures the data itself remains unreadable without the proper key. By walking through both denied and granted access scenarios, I reinforced how encryption adds a critical safeguard beyond traditional access controls. This hands-on practice strengthened my understanding of cloud security fundamentals and prepared me to apply these techniques in real-world environments.
