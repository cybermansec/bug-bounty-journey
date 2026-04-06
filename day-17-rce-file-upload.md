##  Vulnerability Report: Remote Code Execution via Unrestricted File Upload

###  Summary

A remote code execution (RCE) vulnerability exists in the profile image upload functionality. The application fails to properly validate uploaded files, allowing an attacker to upload and execute a malicious server-side script.

---

###  Vulnerable endpoint

`POST /my-account/avatar`

---

###  Description

The application allows users to upload profile images. However the server does not adequately validate the file type or restrict execution of uploaded files.

An attacker can upload a malicious script (i.e, a PHP file) disguised as an image. Once uploaded, the file is stored in a web-accessible directory and executed by the server when accessed via a browser.

This leads to **remote code execution**, allowing an attacker to run arbitrary commands on the server.

---

###  Steps to Reproduce

1. Log in to a valid user account.
2. Navigate to the profile image upload feature.
3. Intercept the upload request using a proxy tool.
4. Modify the request:

   * Change the filename to a server-executable extension (i.e, `.php`)
   * Replace the file content with a malicious script
5. Forward the modified request to the server.
6. Locate the uploaded file via its accessible URL.
7. Access the file in a browser to trigger execution.

---

###  Proof of Concept (Impact Demonstration)

A malicious script was uploaded that executed server-side code to retrieve sensitive data (a user secret). Accessing the uploaded file successfully executed the code and returned the sensitive information.

This confirms that arbitrary code execution is possible on the server.

---

###  Impact

This vulnerability allows an attacker to:

* Execute arbitrary code on the server
* Access sensitive files and data
* Potentially compromise the entire application
* Escalate to full server takeover depending on environment configuration

Severity: **Critical**

---

###  Remediation Recommendations

To mitigate this issue, the following should be implemented:

* Strictly validate file types using a whitelist (i.e, only allow `.jpg`, `.png`)
* Verify file content (not just extension or MIME type)
* Rename uploaded files to random, non-executable names
* Store uploaded files outside the web root
* Disable execution in upload directories (i.e, via server configuration)
* Implement proper access controls on uploaded files

---

###  Additional Notes

This vulnerability demonstrates a complete breakdown in file upload validation and execution controls. Immediate remediation is strongly recommended due to the severity of the issue.

---
