+++
title = "Modernising Your Authentication"
date = 2021-03-16T10:21:37Z
images = []
tags = []
categories = []
draft = true
+++

## Background

Getting started with Authentication in .NET Core 5 is dead easy. An entire authentication stack can be set up in a few clicks using the templates provided by Microsoft and it's pretty comprehensive. 

If there's an existing authentication system then things can get a bit tricky. The scenario that I'll discuss today is how to integrate an existing legacy authentication stack into a new .NET Core API project so that we can issue JWT tokens to our API consumers while avoiding having two competing authentication systems. _complete a smaller work item and avoid doing more work_.


## Requiring Authenticated Users

Forcing .NET Core to only accept requests from authenticated users takes a few lines of code:

#### **`startup.cs`**
```
services.AddMvc(options => 
    options.Filters.Add(new AuthorizeFilter(
        new AuthorizationPolicyBuilder().RequireAuthenticatedUser().Build()
    ))
);
```

I'm only covering JWT-based authentication in this article, but modifying this setup to authenticate users using cookies or other authentication methods should be simple.

To enable JWT authentication:

#### **`Startup.cs`**
```
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidAudience = <value from config>,
        ValidIssuer = <value from config>,
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = <value from config>
    };
});
```
You will probably want to use a X509 certificate to sign your JWTs and prove their authenticity.

## Adding Our Authentication System

Now we are requiring JWTs to be used to authenticate our API, we just need a way of creating them. Let's give our system an AuthenticationService which will handle actually authenticating a user.

#### **`AuthenticationService.cs`**
```
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidAudience = <value from config>,
        ValidIssuer = <value from config>,
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = <value from config>
    };
});
```

