[open-issues]: https://github.com/it-at-m/keycloak-custom-otp-plugin/issues
[new-issue]: https://github.com/it-at-m/keycloak-custom-otp-plugin/issues/new/choose
[license]: ./LICENSE
[new-issue-shield]: https://img.shields.io/badge/new%20issue-blue?style=for-the-badge
[made-with-love-shield]: https://img.shields.io/badge/made%20with%20%E2%9D%A4%20by-it%40M-yellow?style=for-the-badge
[license-shield]: https://img.shields.io/github/license/it-at-m/refarch?style=for-the-badge
[itm-opensource]: https://opensource.muenchen.de/

# keycloak-custom-otp-plugin

[![New issue][new-issue-shield]][new-issue]
[![Made with love by it@M][made-with-love-shield]][itm-opensource]
[![GitHub license][license-shield]][license]

TODO

## Built With

- OpenJDK 21
- Keycloak 26

## Installation
- start Keycloak
- Build and deploy:
    - cd backend
    - mvn clean install
    - run locally with docker:
        - `docker build . -t keycloak-custom-otp`
        - `docker run --name keycloak-custom-otp -p 8443:8443 keycloak-custom-otp start-dev`

## Configuration:
- Login to Keycloak as admin
- create new realm `safe`
- realm settings --> Themes --> set login theme to `otptheme`
- realm settings --> Localization --> Set Internationalization to enabled, supported locales "German" and "English"
- Authentication --> Flows
    - Browser-flow -->  Duplicate with name `browser_custom_otp`
    - Configure like this
      ![browser_custom_otp](browser_custom_otp.png)
    - Button "Action" --> Bind flow
- Creat neuen user otp (PW: otp) (firstname, lastname and eMail necessary but anything)
- In realm master: Clients --> Client safe-realm, tab Roles, button "create role", create role with name "custom-add-otp"
- Users --> User "admin" --> Tab "Role mapping"
    - Button "Assign role", find role "custom-add-otp" and assign
    - Button "Assign role" --> search for "safe-realm" --> assign role "manage-users"

## Test
- Check user in admin console:
    - https://localhost:8443/auth/admin/master/console/#/safe/users
    - User otp
- first login:
    - https://localhost:8443/auth/realms/safe/account
    - User: otp/otp
    - should show error message that OTP is not configured
- rollout OTP via POST-request:
    - get token for admin:
        - POST https://localhost:8443/auth/realms/master/protocol/openid-connect/token
        - Header: Content-Type application/x-www-form-urlencoded
        - Body: client_id=admin-cli&grant_type=password&username=admin&password=admin
        - execute, copy access token
    - create TOTP for user
        - POST http://localhost:8080/auth/realms/safe/totp
        - Header: Content-Type application/json
        - Header: Authorization bearer <token from Call 1>
        - Body: `{"username": "otp"}`
        - should result in:
      ```
      {
      "secret": "gram6k9gm2kqstLoqR9x",
      "secretQrcode": "iVBORw...kJggg=="
      }
      ```
- Copy content of secretQrcode into file backend/src/main/resources/ViewQRCode.html
  (aber `base64, ` up to the closing quotes)
- open ViewQrCode.html in browser
- Scan in smartphone with OTP App like FreeOTP or Google Authenticator
- Second login:
    - this time it asks for OTP
    - account login page shows
- remove TOTP via DELETE-request
    - get token for admin (see above)
    - remove TOTP
        - DELETE https://localhost:8443/auth/realms/safe/totp/otp
        - Header: Authorization bearer <token from Call 1>
        - no body

## Roadmap

See the [open issues][open-issues] for a full list of proposed features (and known issues).

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please open an issue with the tag "enhancement", fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Open an issue with the tag "enhancement"
2. Fork the Project
3. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
4. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
5. Push to the Branch (`git push origin feature/AmazingFeature`)
6. Open a Pull Request

More about this in the [CODE_OF_CONDUCT](./.github/CODE_OF_CONDUCT.md) file.

## License

Distributed under the MIT License. See [LICENSE](LICENSE) file for more information.

## Contact

it@M - <opensource@muenchen.de>
