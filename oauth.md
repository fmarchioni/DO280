```mermaid
graph TD
    User[User] -->|Request Login| OauthServer[OAuth Server]
    OauthServer -->|Redirect| User
    User -->|Provide Credentials| IdentityProvider[Identity Provider]
    IdentityProvider -->|Authenticate| OauthServer
    OauthServer -->|Issue Token| User
    User -->|Access Resources| OpenShiftAPI[OpenShift API]
    OpenShiftAPI -->|Verify Token| OauthServer
    OpenShiftAPI -->|Grant Access| User
