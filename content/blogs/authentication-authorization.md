---
title: "Implementing Authentication and Authorization in Go"
date: 2024-07-01T00:00:00-07:00
draft: false
github_link: "https://github.com/bindrad/bindrad.github.io"
author: "Dhruv Bindra"
tags:
  - Authentication
  - Authorization
  - Golang
image: /images/auth-go.jpeg
description: ""
toc: 
---

When building secure web applications, authentication and authorization are critical components that ensure users have appropriate access to resources. In this blog, we'll explore how to implement both using Go, focusing on practical examples and best practices.

## Authentication vs. Authorization

Before diving into the implementation, it's essential to distinguish between authentication and authorization:

- Authentication: Verifies the identity of a user (e.g., logging in with a username and password).
- Authorization: Determines whether an authenticated user has permission to access specific resources.

## Setting Up the Project

First, let's set up a basic Go project. We'll use `github.com/gin-gonic/gin` for the web framework and `github.com/golang-jwt/jwt/v4` for handling JSON Web Tokens (JWTs).


- Initialize a new Go module:

```
go mod init auth-demo
go get github.com/gin-gonic/gin
go get github.com/golang-jwt/jwt/v4
```

- Create the main application file:

```
package main

import (
    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v4"
    "net/http"
    "fmt"
    "time"
)

func main() {
    r := gin.Default()
    
    r.POST("/login", loginHandler)
    r.GET("/secure", authMiddleware(), secureHandler)
    
    r.Run(":8080")
}
```

## Implementing Authentication

We'll create an endpoint to handle user login. This endpoint will validate user credentials and issue a JWT if successful.

```
func loginHandler(c *gin.Context) {
    var loginData struct {
        Username string `json:"username"`
        Password string `json:"password"`
    }
    
    if err := c.ShouldBindJSON(&loginData); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid request"})
        return
    }

    // Replace with your user authentication logic
    if loginData.Username == "admin" && loginData.Password == "password" {
        token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
            "username": loginData.Username,
            "role":     "admin",
            "exp":      time.Now().Add(time.Hour * 72).Unix(),
        })
        
        tokenString, err := token.SignedString([]byte("your-secret-key"))
        if err != nil {
            c.JSON(http.StatusInternalServerError, gin.H{"error": "Could not generate token"})
            return
        }
        
        c.JSON(http.StatusOK, gin.H{"token": tokenString})
    } else {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
    }
}
```

## Implementing Authorization

Next, we'll implement middleware to authorize users based on their roles. This middleware will parse the JWT, validate it, and ensure the user has the appropriate role to access the resource.

```
func authMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        tokenString := c.GetHeader("Authorization")
        if tokenString == "" {
            c.AbortWithStatus(http.StatusUnauthorized)
            return
        }
        
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
            }
            return []byte("your-secret-key"), nil
        })
        
        if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
            c.Set("claims", claims)
        } else {
            c.AbortWithStatus(http.StatusUnauthorized)
            return
        }
        
        c.Next()
    }
}

func secureHandler(c *gin.Context) {
    claims := c.MustGet("claims").(jwt.MapClaims)
    role := claims["role"].(string)
    
    if role != "admin" {
        c.AbortWithStatus(http.StatusForbidden)
        return
    }
    
    c.JSON(http.StatusOK, gin.H{"message": "Welcome, admin!"})
}
```

## Implementing Role-Based Access Control

For a more advanced authorization system, we'll handle different roles and permissions. The following example checks user roles and permissions before allowing access to an endpoint.

```
func roleBasedAuthMiddleware(requiredRole string) gin.HandlerFunc {
    return func(c *gin.Context) {
        claims := c.MustGet("claims").(jwt.MapClaims)
        role := claims["role"].(string)
        
        if role != requiredRole {
            c.AbortWithStatus(http.StatusForbidden)
            return
        }
        
        c.Next()
    }
}

r.GET("/admin", authMiddleware(), roleBasedAuthMiddleware("admin"), adminHandler)
```

## Conclusion

Implementing authentication and authorization in Go is straightforward with the help of packages like gin-gonic/gin and github.com/golang-jwt/jwt/v4. By following best practices and properly managing roles and permissions, you can secure your application effectively. Remember to keep your secret keys secure and follow the principle of least privilege when designing your authorization logic.

By following these steps, you can build a secure and robust authentication and authorization system in your Go applications.
