# OIDC - OpenID Connect

## State of the art (2024)

- OUT: Client-side packages in Angular
- IN: BFF - Angular app: Login in Backend (read stuff from Damien Bowden) 

## HTTPS

### Issue: redirects to http instead of https

I found several options:

- Tested, works. Set `ASPNETCORE_FORWARDEDHEADERS_ENABLED` to true: <[https://stackoverflow.com/questions/50468033/redirect-uri-sent-as-http-and-not-https-in-app-running-https/76047542#76047542](https://stackoverflow.com/a/76047542/8035608)>
- Tested, works. Set `context.ProtocolMessage.RedirectUri=...` in `OnRedirectToIdentityProvider` <https://stackoverflow.com/a/68410484>
- not tested, might help: <https://stackoverflow.com/questions/50468033/redirect-uri-sent-as-http-and-not-https-in-app-running-https>
- not tested, might help: <https://stackoverflow.com/questions/72488243/openidconnect-redirects-to-http-instead-of-https>

### Avoid https redirection

Better remove `app.UseHttpsRedirection();` because this will not forward authorization headers from http to https-

## How to avoid Login Page

Redirect url `prompt=none`
