# OIDC - OpenID Connect

## State of the art (2024)

- OUT: Client-side packages in Angular
- IN: BFF - Angular app: Login in Backend (read stuff from Damien Bowden) 

## Avoid https redirection

Better remove `app.UseHttpsRedirection();` because this will not forward authorization headers from http to https-

## How to avoid Login Page

Redirect url `prompt=none`
