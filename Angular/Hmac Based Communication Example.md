
# HMAC communcation example

Here is the pre-defined request header based on hmac for communication between frontend (angular) backend(java)

**Request header format**
```
Authorization: hmac app-id:signature:nonce:ts
```
 - app-id: a unique id assigned to the mobile application as consumer of the API 
 - app-secret: a shared between the server and the mobile  applicaition secret string
 - nonce: an arbitrary number/string used only once
 - ts: the number of seconds since January 1, 1970 00:00:00  GMT

**Request-string**
a string constructing from the following parameters in this order:
 - app-id
 - HTTP method (lower case)
 - request URI (lower case)
 - ts
 - nonce
 - base64-encoded string representation of the request payload
Each parameter must be trimmed from heading and trailing spaces

**Signature**
calculated using HMAC with the specified hash algorithm "sha256" and the app-secret over the normalized request-string and base64-encoded

**Response header format**
```
Server-Authorization: hmac signature
```
**Response-string**
a string constructing from the following parameters in this order:
 - app-id
 - HTTP method (lower case)
 - request URI (lower case)
 - ts
 - nonce
 - base64-encoded string representation of the request payload
Each parameter must be trimmed from heading and trailing spaces

**Signature**
calculated using HMAC with the specified hash algorithm "sha256" and the app-secret over the normalized request-string and base64-encoded

Sure! Below is an example of how you can set up end-to-end communication between an Angular frontend and a Java Spring MVC backend using the HMAC authorization header.

### Frontend (Angular)

1. **Install Axios**: First, ensure you have Axios installed for making HTTP requests.
   ```bash
   npm install axios
   ```

2. **Create a Service to Handle Requests**:
   ```typescript
   import { Injectable } from '@angular/core';
   import { HttpClient, HttpHeaders } from '@angular/common/http';
   import * as CryptoJS from 'crypto-js';

   @Injectable({
     providedIn: 'root'
   })
   export class ApiService {
     private apiUrl = 'http://localhost:8080/api/data';

     constructor(private http: HttpClient) {}

     async makeRequest(method: string, url: string, payload?: any): Promise<any> {
       const appId = 'your-app-id';
       const appSecret = 'your-app-secret';
       const nonce = Math.random().toString(36).substr(2, 15);
       const ts = Math.floor(Date.now() / 1000);

       let requestString = `${appId.toLowerCase()}${method.toLowerCase()}${url.toLowerCase()}${ts}${nonce}`;
       if (payload) {
         payload = JSON.stringify(payload).trim();
         requestString += CryptoJS.enc.Base64.stringify(CryptoJS.enc.Utf8.parse(payload));
       }

       const signature = CryptoJS.HmacSHA256(requestString, appSecret).toString(CryptoJS.enc.Hex);

       const headers = new HttpHeaders({
         'Authorization': `hmac ${appId}:${signature}:${nonce}:${ts}`
       });

       let response;
       try {
         if (method === 'get') {
           response = await this.http.get<any>(this.apiUrl + url, { headers }).toPromise();
         } else if (method === 'post') {
           response = await this.http.post<any>(this.apiUrl + url, payload, { headers }).toPromise();
         } else if (method === 'put') {
           response = await this.http.put<any>(this.apiUrl + url, payload, { headers }).toPromise();
         } else if (method === 'delete') {
           response = await this.http.delete<any>(this.apiUrl + url, { headers }).toPromise();
         }
       } catch (error) {
         console.error('Error:', error);
         throw error;
       }

       return response;
     }
   }
   ```

3. **Using the Service in a Component**:
   ```typescript
   import { Component } from '@angular/core';
   import { ApiService } from './api.service';

   @Component({
     selector: 'app-root',
     template: `<div>{{ data }}</div>`
   })
   export class AppComponent {
     data: any;

     constructor(private apiService: ApiService) {}

     async ngOnInit() {
       try {
         const response = await this.apiService.makeRequest('get', '/data');
         this.data = response;
       } catch (error) {
         console.error('Error fetching data:', error);
       }
     }
   }
   ```

### Backend (Spring MVC)

1. **Add Dependencies**: Ensure your `pom.xml` includes the necessary dependencies for Spring Boot and HMAC.
   ```xml
   <dependencies>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt-api</artifactId>
       <version>0.11.5</version>
     </dependency>
   </dependencies>
   ```

2. **Create a Controller to Handle Requests**:
   ```java
   import org.springframework.web.bind.annotation.*;

   @RestController
   @RequestMapping("/api/data")
   public class DataController {

       @GetMapping
       public String getData() {
           return "Hello, World!";
       }

       @PostMapping
       public String postData(@RequestBody String payload) {
           return "Received: " + payload;
       }
   }
   ```

3. **Create a Filter to Verify HMAC Signature**:
   ```java
   import org.springframework.stereotype.Component;
   import javax.servlet.*;
   import javax.servlet.http.HttpServletRequest;
   import java.io.IOException;
   import java.security.InvalidKeyException;
   import java.security.NoSuchAlgorithmException;
   import java.util.Base64;
   import javax.crypto.Mac;
   import javax.crypto.spec.SecretKeySpec;

   @Component
   public class HmacAuthFilter implements Filter {

       private static final String APP_SECRET = "your-app-secret";

       @Override
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
               throws IOException, ServletException {
           HttpServletRequest httpRequest = (HttpServletRequest) request;
           String authorizationHeader = httpRequest.getHeader("Authorization");

           if (authorizationHeader == null || !authorizationHeader.startsWith("hmac ")) {
               HttpServletResponse httpResponse = (HttpServletResponse) response;
               httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
               return;
           }

           String[] parts = authorizationHeader.substring(4).split(":");
           if (parts.length != 4) {
               HttpServletResponse httpResponse = (HttpServletResponse) response;
               httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
               return;
           }

           String appId = parts[0];
           String signature = parts[1];
           String nonce = parts[2];
           long ts = Long.parseLong(parts[3]);

           // Validate nonce and timestamp
           if (!isValidNonceAndTimestamp(nonce, ts)) {
               HttpServletResponse httpResponse = (HttpServletResponse) response;
               httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
               return;
           }

           // Reconstruct request string
           String method = httpRequest.getMethod().toLowerCase();
           String uri = httpRequest.getRequestURI().toLowerCase();
           String requestString = appId + method + uri + ts + nonce;

           try {
               byte[] appSecretBytes = APP_SECRET.getBytes();
               SecretKeySpec secretKey = new SecretKeySpec(appSecretBytes, "HmacSHA256");
               Mac mac = Mac.getInstance("HmacSHA256");
               mac.init(secretKey);
               String calculatedSignature = Base64.getEncoder().encodeToString(mac.doFinal(requestString.getBytes()));

               if (!calculatedSignature.equals(signature)) {
                   HttpServletResponse httpResponse = (HttpServletResponse) response;
                   httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                   return;
               }
           } catch (NoSuchAlgorithmException | InvalidKeyException e) {
               HttpServletResponse httpResponse = (HttpServletResponse) response;
               httpResponse.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
               return;
           }

           chain.doFilter(request, response);
       }

       private boolean isValidNonceAndTimestamp(String nonce, long ts) {
           // Implement your logic to validate nonce and timestamp
           // For example, check if the nonce has been used before or if the timestamp is within a reasonable range
           return true; // Placeholder
       }
   }
   ```

4. **Configure Filter in Spring Boot**:
   ```java
   import org.springframework.boot.web.servlet.FilterRegistrationBean;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;

   @Configuration
   public class FilterConfig {

       @Bean
       public FilterRegistrationBean<HmacAuthFilter> hmacAuthFilter() {
           FilterRegistrationBean<HmacAuthFilter> registrationBean = new FilterRegistrationBean<>();
           registrationBean.setFilter(new HmacAuthFilter());
           registrationBean.addUrlPatterns("/api/*");
           registrationBean.setOrder(1);
           return registrationBean;
       }
   }
   ```

This setup demonstration how securely communicate between Angular frontend and Java Spring MVC backend using the HMAC authorization header.