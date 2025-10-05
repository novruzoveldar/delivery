# 🛡️ Keycloak Authentication Setup — Delivery Platform

This guide explains how to configure and integrate **Keycloak** as the **Identity and Access Management (IAM)** provider for the **Guavapay Delivery Microservices** ecosystem.
Keycloak provides a secure and centralized way to handle user authentication, authorization, and token management across all services (Gateway, Admin, Courier, Parcel, etc.).

---

## 🧭 Overview

**Keycloak** is an open-source Identity and Access Management tool for modern applications and services.
It handles:

* **Single Sign-On (SSO)** across microservices
* **JWT/OAuth2 token issuance**
* **User management and roles**
* **Integration with external identity providers**

In the **Delivery Platform**, Keycloak secures access to:

* API Gateway (`gw-delivery`)
* Admin panel (`ms-delivery-admin`)
* Courier order service (`ms-courier-order`)
* Parcel order service (`ms-parcel-order`)

---

## 🧱 Realm Configuration

A dedicated realm called **`Delivery`** is created to manage all authentication and authorization aspects of the platform.

### Realm Details

| Setting              | Value                                       |
| -------------------- | ------------------------------------------- |
| **Realm Name**       | `Delivery`                                  |
| **Display Name**     | Delivery                                    |
| **Enabled**          | ✅ Yes                                       |
| **Login with Email** | ✅ Enabled                                   |
| **User Management**  | Enabled                                     |
| **Endpoints**        | [Realm OpenID Endpoints](#openid-endpoints) |

---

## 🔐 OpenID Endpoints

Once the realm is created, Keycloak exposes the following OpenID Connect (OIDC) discovery endpoint:

```
http://<keycloak-host>:8080/realms/Delivery/.well-known/openid-configuration
```

Example (local setup):

```
http://localhost:8080/realms/Delivery/.well-known/openid-configuration
```

This endpoint provides metadata about token, authorization, and user info URLs that your microservices will consume.

---

## ⚙️ Realm Setup Steps

1. **Login to Keycloak Admin Console**
   Visit: [http://localhost:8080](http://localhost:8080)
   Default credentials (if not changed):

   ```
   Username: admin
   Password: admin
   ```

2. **Create Realm**

   * Click on the top-left realm dropdown → **Add Realm**
   * Realm name: `Delivery`
   * Click **Create**

3. **Configure Realm Settings**

   * Display name: `Delivery`
   * Enabled: ✅
   * User management: ✅
   * Save changes

4. **Create Clients**

   * Go to: **Clients → Create**
   * Example clients:

     | Client ID           | Access Type  | Root URL                                       | Valid Redirect URIs |
     | ------------------- | ------------ | ---------------------------------------------- | ------------------- |
     | `gw-delivery`       | public       | [http://localhost:8080](http://localhost:8080) | `/*`                |
     | `ms-delivery-admin` | confidential | [http://localhost:8081](http://localhost:8081) | `/*`                |
     | `ms-courier-order`  | confidential | [http://localhost:8082](http://localhost:8082) | `/*`                |
     | `ms-parcel-order`   | confidential | [http://localhost:8083](http://localhost:8083) | `/*`                |

5. **Add Roles**

   * Go to **Roles → Add Role**
   * Create:

     * `ADMIN`
     * `COURIER`
     * `CUSTOMER`

6. **Create Users**

   * Go to **Users → Add User**
   * Add credentials under the **Credentials** tab
   * Assign roles under **Role Mappings**

7. **Configure Tokens**

   * Under **Realm Settings → Tokens**, set:

     * Access Token Lifespan: `1h`
     * Refresh Token Lifespan: `12h`

8. **Integration with Microservices**

   * Update each microservice’s configuration (`application.yml` or `bootstrap.yml`):

     ```yaml
     spring:
       security:
         oauth2:
           resourceserver:
             jwt:
               issuer-uri: http://localhost:8080/realms/Delivery
     ```

---

## 🧩 Example Login Flow

1. A user accesses the Admin Panel.
2. The frontend redirects to Keycloak’s login page.
3. After authentication, Keycloak issues a **JWT token**.
4. The token is passed via the `Authorization: Bearer <token>` header to downstream services.
5. Services verify the token against the Keycloak realm configuration.

---

## 📸 Interface Overview

| Login Screen                                                                                | Realm Configuration                                                                           |
| ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| ![Keycloak Login Screen](https://www.keycloak.org/resources/images/keycloak_icon_512px.svg) | ![Realm Config Example](https://www.keycloak.org/resources/images/keycloak_admin_console.png) |

*(Note: Replace above URLs with your hosted screenshots if you’d like to show your actual instance.)*

---

## 🧠 Troubleshooting

| Issue                           | Cause                                      | Solution                                 |
| ------------------------------- | ------------------------------------------ | ---------------------------------------- |
| `401 Unauthorized`              | Invalid or expired JWT token               | Reauthenticate via Keycloak              |
| `Invalid token issuer`          | Wrong issuer URI in microservice config    | Match `issuer-uri` with realm’s endpoint |
| Keycloak login page not loading | Keycloak service down                      | Restart Keycloak container or service    |
| CORS issues                     | Frontend not listed in Valid Redirect URIs | Add frontend base URL in client settings |

---

## 🧰 Useful Commands (Docker)

If running Keycloak via Docker:

```bash
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_USER=admin \
  -e KEYCLOAK_PASSWORD=admin \
  quay.io/keycloak/keycloak:legacy
```

Or for Keycloak 21+ (Quarkus):

```bash
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:latest \
  start-dev
```

---

## 🧾 Summary

| Component          | Description                                                           |
| ------------------ | --------------------------------------------------------------------- |
| **Realm**          | Delivery                                                              |
| **Protocol**       | OpenID Connect (OIDC)                                                 |
| **Authentication** | JWT                                                                   |
| **Integration**    | Spring Boot (OAuth2 Resource Server)                                  |
| **Purpose**        | Centralized login and token management for all delivery microservices |

---



<img width="452" height="158" alt="image" src="https://github.com/user-attachments/assets/d10ab784-35ed-4ccc-bc0c-6626047614be" />

<img width="452" height="177" alt="image" src="https://github.com/user-attachments/assets/8d8a116a-1ca1-4f9e-a2dd-68c7f3087528" />

<img width="452" height="187" alt="image" src="https://github.com/user-attachments/assets/ada24f56-d9fc-4be0-b378-17fce81241d2" />

<img width="833" height="782" alt="Screenshot 2025-10-05 at 21 34 40" src="https://github.com/user-attachments/assets/1fccfac4-a910-407e-b391-fdebd914e024" />



