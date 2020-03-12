# Simple Wallet RESTful API

The name of the project is pretty much descriptive enough. To add a little more on the name, the application does the
stuff you need from a digital wallet i.e.
   1. Registration and Login
   2. Account Deposit
   3. Account Withdrawal
   4. Balance Enquiry
   5. Mini Statement
   
PS. This is fully a backend application with no front end client whatsoever. You can use curl, postman or your favourite
http tool to interact with the application.

You can follow the below steps to install and setup the application.

## Installation

To begin with the application uses postgres as the backend database.

### Database Installation

The application uses postgres as the database server. So here are instructions on how to setup postgresql
on your machine using docker.

Get the official postgres docker image.
```bash
$ docker pull postgres
``` 

Then create a container from the image with the following variables
```bash
$ docker create \
--name wallet-db \
-e POSTGRES_USER=wallet \
-e POSTGRES_PASSWORD=wallet \
-p 5432:5432 \
postgres
```

Run the following command to start the container
```bash
$ docker start wallet-db
```

### Simple Wallet Installation

Running the application is as simple as running any other go application but first we need to copy and create our
configuration.

#### Lets begin. Cloning ...

```bash
$ git clone https://github.com/SirWaithaka/simple-wallet.git
```

#### Configuring
```bash
$ cd simple-wallet
$ cp config.yml.example config.yml
```

This configuration file looks something like this

```yaml
database:
  host: "127.0.0.1"
  port: "5432"
  user: "wallet"
  password: "wallet"
  dbname: "wallet"

app_secret_key: "eQig7GS4cHO2su"
```

You can change the config variables depending on your database setup, here i choose to follow the default setup shown at
database installation step.

#### Building and running

```bash
$ cd main
$ go build wallet-server.go
$ ./wallet-server
```

It will install all dependencies required and produce a binary for you platform. Then the server will start at port
`6700`.

Enjoy.

## API

A description of the api.

### Endpoints

All the routes exposed in the application are all defined in this function
```go
func apiRouteGroup(g *fiber.Group, domain *registry.Domain, config wallet.Config) {
	g.Post("/login", users.Authenticate(domain.User))
	g.Post("/user", users.Register(domain.User))

	g.Get("/account/balance", middleware.AuthByBearerToken(config.Secret), accounts.BalanceEnquiry(domain.Account))
	g.Post("/account/deposit", middleware.AuthByBearerToken(config.Secret), accounts.Deposit(domain.Account))
	g.Post("/account/withdrawal", middleware.AuthByBearerToken(config.Secret), accounts.Withdraw(domain.Account))

	g.Get("/account/statement", middleware.AuthByBearerToken(config.Secret), accounts.MiniStatement(domain.Transaction))
}
```

The about routes are mounted on the prefix `/api` so your request should point to
```
POST /api/login
POST /api/user # for registration
GET /api/account/balance
POST /api/account/deposit
POST /api/account/withdrawal
GET /api/account/statement
```

#### To Register
A user can be registered to the api with the following `POST` parameters
`firstName`, `lastName`, `email`,  `phoneNumber`, `password`

Response example
```json
{
    "status": "success",
    "message": "user created",
    "user": {
        "email": "newme@email.com",
        "userId": "b4b00501-ba22-49fb-827d-b25d969c58bb"
    }
}
```

#### To Login
You can use `email and password` or `phoneNumber and password`

Response example

```json
{
    "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7InVzZXJJZCI6Ijg0ODA5YTAyLTkwODItNGFlMy05MDQ3LTM4NDA5NDhjNTdjZiIsImVtYWlsIjoiaGFsbEBlbWFpbC5jb20ifSwiZXhwIjoxNTg0MDI0NTQ0LCJpYXQiOjE1ODQwMDI5NDR9.qZHLJWtYK7_ClgnaPJbGuaiPW8ssd1Ra9xFJWdg6iwE"
}
```

The remaining endpoints require the token acquired above for authentication

#### To Deposit
You only need the `amount` parameter

Response example

```json
{
    "message": "Amount successfully deposited new balance 5000",
    "balance": 5000,
    "userId": "84809a02-9082-4ae3-9047-3840948c57cf"
}
``` 

#### To Withdraw
You only need the `amount` parameter

Response example

```json
{
    "message": "Amount successfully withdrawn new balance 4700",
    "balance": 4700,
    "userId": "84809a02-9082-4ae3-9047-3840948c57cf"
}
```

#### To Query Balance
This is just a `GET` request, no params

Response example

```json
{
    "message": "Your current balance is 4700",
    "balance": 4700
}
```

#### To Get Mini Statement
This is just a `GET` request, no params

Response example

```json
{
    "message": "ministatement retrieved for the past 5 transactions",
    "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
    "transactions": [
        {
            "transactionId": "1088b880-1aa1-4cf0-929b-dfab01c52c13",
            "transactionType": "balance_enquiry",
            "timestamp": "2020-03-12T13:11:09.863693Z",
            "amount": 4700,
            "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
            "accountId": "fd1ce4e4-e467-4eac-8ea0-ea0c9d4f76fe"
        },
        {
            "transactionId": "8b64ca58-b869-47b6-964a-0846957d4c7f",
            "transactionType": "withdrawal",
            "timestamp": "2020-03-12T12:30:37.355034Z",
            "amount": 4700,
            "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
            "accountId": "fd1ce4e4-e467-4eac-8ea0-ea0c9d4f76fe"
        },
        {
            "transactionId": "4514a94c-f303-4324-b010-a4e7c3dd3f77",
            "transactionType": "withdrawal",
            "timestamp": "2020-03-12T12:30:36.278053Z",
            "amount": 4710,
            "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
            "accountId": "fd1ce4e4-e467-4eac-8ea0-ea0c9d4f76fe"
        },
        {
            "transactionId": "871035ad-456a-4467-8260-414b464b6d86",
            "transactionType": "withdrawal",
            "timestamp": "2020-03-12T12:30:35.446227Z",
            "amount": 4720,
            "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
            "accountId": "fd1ce4e4-e467-4eac-8ea0-ea0c9d4f76fe"
        },
        {
            "transactionId": "c3c3a19a-fc3a-4080-9b52-5d3e19853cd7",
            "transactionType": "withdrawal",
            "timestamp": "2020-03-12T12:30:34.646326Z",
            "amount": 4730,
            "userId": "84809a02-9082-4ae3-9047-3840948c57cf",
            "accountId": "fd1ce4e4-e467-4eac-8ea0-ea0c9d4f76fe"
        }
    ]
}
```
