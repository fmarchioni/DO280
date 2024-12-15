```mermaid
graph TD
    User[User]:::user -->|Request Login| OauthServer[OAuth Server]:::oauth
    OauthServer -->|Redirect| User
    User -->|Provide Credentials| IdentityProvider[Identity Provider]:::idp
    IdentityProvider -->|Authenticate| OauthServer
    OauthServer -->|Issue Token| User
    User -->|Access Resources| OpenShiftAPI[OpenShift API]:::api
    OpenShiftAPI -->|Verify Token| OauthServer
    OpenShiftAPI -->|Grant Access| User

    classDef user fill:#FFFFFF,stroke:#000,stroke-width:2px;
    classDef oauth fill:#1E90FF,stroke:#000,stroke-width:2px;
    classDef idp fill:#32CD32,stroke:#000,stroke-width:2px;
    classDef api fill:#FF4500,stroke:#000,stroke-width:2px;

```



# Flusso di Funzionamento:

* L'utente accede a un'applicazione e viene reindirizzato all'OAuth Server per autenticarsi.
* L'OAuth Server reindirizza l'utente all'Identity Provider per inserire le credenziali.
* Dopo l'autenticazione, l'Identity Provider comunica l'esito all'OAuth Server.
* L'OAuth Server emette un token di accesso e lo restituisce all'utente.
* L'utente utilizza il token per effettuare richieste all'API di OpenShift.
* L'API verifica il token con l'OAuth Server prima di concedere l'accesso.
