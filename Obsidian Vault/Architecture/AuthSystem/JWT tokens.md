## Intro
### What Is 
[JSON Web Tokens](https://jwt.io/introduction) are an open, industry standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties.<br>
JWT.IO allows you to decode, verify and generate JWT.<br>

### When should you use JSON Web Tokens?
-> **Authorization**: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.<br>
-> **Information Exchange**: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.
### What is the JSON Web Token structure?
In its compact form, [JSON Web Tokens](https://jwt.io/introduction) consist of three parts separated by dots (`.`), which are:<br>
-> Header<br>
-> Payload<br>
-> Signature<br>
## How to use
### Library to use
Here [desc](https://jwt.io/libraries?language=JavaScript)<br>
-> jose -> `npm install jose`<br>
-> jsrsasign -> `npm install jsrsasign`<br>
-> jose-jwe-jws -> `npm install jose-jwe-jws`<br>
-> node-jose -> `npm install node-jose`<br>