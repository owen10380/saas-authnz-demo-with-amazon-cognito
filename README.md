# SaaS AuthN/Z Demo with Amazon Cognito

The SaaS AuthN/Z Demo explains concepts related to authentication and authorization in SaaS products on AWS, and provides a sample application implemented with Amazon Cognito.

**Disclaimer:** This application is intended for educational purposes and is designed to be used within the context of the [SaaS AuthN/Z Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/9180bbda-7747-4b8f-ac05-14e7f258fcea). It aims to teach the functionalities of Amazon Cognito, Amazon Verified Permissions, and design patterns for SaaS. It is not intended for production use. If you plan to use parts of the code or assets in this repository in your own product, please perform thorough consideration beforehand.

## Table of Contents
* [Concept](#concept)
* [Deployment](#deployment)
* [Architecture](#architecture)
* [Scenarios](#scenarios)
  * [Execution of Onboarding Process](/docs/onboarding.md)
  * [Sign-in to Tenant](/docs/sign-in.md)
  * [Management of Tenant and Users](/docs/manage-tenant-and-users.md)
    * [Authorization Using JWT Attributes](/docs/authorize.md)
    * [Details of Sign-in with External IdP](/docs/federation-signin.md)
* [Environment Deletion](#environment-deletion)
* Miscellaneous
  * [Details of Tenant Service](/docs/tenant-service.md)
  * [Multi-Tenancy Strategy with Amazon Cognito](/docs/cognito-multi-tenancy.md)

## Concept

Implementing authentication and authorization in B2B SaaS applications requires additional considerations compared to regular web application authentication and authorization. For instance, depending on the security policies of the client companies, there might be a need to provide authentication options, and features for administrators of tenants to manage other users. The backend application needs to handle access control based on various elements such as tenant isolation, contract plans, user roles, etc. Efficient handling of information scattered across various databases and microservices for authorization is also crucial.

![AccessPattern](/docs/images/saas-access-control-pattern.en.png)

In access control, as introduced in [this article](https://aws.amazon.com/jp/builders-flash/202108/saas-authorization-implementation-pattern/), one can simplify backend authorization processing by initially including information scattered across various IdPs and databases in a JSON Web Token (JWT) and sending it to the backend. This sample application illustrates how to implement these SaaS authentication concepts using Amazon Cognito and Amazon Verified Permissions.

## Deployment

Follow the [deployment instructions](/docs/how-to-deploy.md) to deploy the application.

## Architecture

![Architecture](/docs/images/architecture.png)

| Service Name | Description |
|--|--|
| Tenant Service | Microservice responsible for core functionalities related to SaaS authentication and tenant management. Refer to [here](/docs/tenant-service.md) for details on the tenant service in this demo application. |
| Identity Service<br> (Amazon Cognito) | Provides authentication and linking functionality between users and tenants. This application adopts an implementation that assigns an application client for each tenant. It supports authentication using email and password, as well as authentication using an external IdP specific to each tenant.<br>**Note:** Within a single user pool, as mentioned in the [resource quotas](https://docs.aws.amazon.com/ja_jp/cognito/latest/developerguide/limits.html#resource-quotas), there are limits on the number of app clients and identity providers. In the demo application, to avoid constraints on the number of tenants due to resource quotas, a new user pool is created for every 300 tenants, which is the default quota for the number of external identity providers per user pool. Refer to [Amazon Cognito Multi-Tenancy](/docs/cognito-multi-tenancy.md) for more details. |
| Frontend Application | Provides authentication screens and user management screens. It offers APIs and screens for authentication, user, and tenant management. The user and tenant management API, after validating the token of the authenticated user, calls the backend tenant service based on the user's permissions. In this demo application, API Gateway uses AWS service integration and mapping templates to call the tenant service directly, narrowing down the scope to the tenant to which the requesting user belongs. |

## Scenarios

The demo application assumes the following scenarios:

1. The operator of the SaaS provider adds a tenant environment and administrator users within the tenant.
2. The tenant service adds administrator users to the Amazon Cognito identity service.
3. The administrator user of the tenant receives an invitation email from Amazon Cognito.
4. The administrator user of the tenant accesses the URL mentioned in the invitation email, displaying the login form of the frontend application. At this point, information on which user pool and application client to use for each tenant is obtained.
5. The administrator user of the tenant authenticates with Amazon Cognito and obtains a token.
6. Upon successful sign-in, the frontend application calls APIs with the acquired token.
7. The Lambda Authorizer and Amazon Verified Permissions verify whether the signed-in user is allowed to make the requested API call.
8. If the API call is permitted, API Gateway calls the backend service with the narrowed-down scope to the tenant. In this application, the API Gateway directly requests the tenant service, but it is common for a Lambda function placed behind API Gateway to call core services of the control plane.

Refer to the following documents for technical details in each phase:

* [**Execution of Onboarding Process**](/docs/onboarding.md): Adds new tenants and users. Corresponds to processes (1) - (3) in the above architecture diagram.
* [**Sign-in to Tenant**](/docs/sign-in.md): Accesses the application as a user of the tenant, signs in to the user pool, and obtains a token with the tenant context. Corresponds to processes (4) - (5) in the above architecture diagram.
* [**Management of Tenant and Users**](/docs/manage-tenant-and-users.md): As an administrator user within the tenant, makes backend API calls to manage users within the tenant. Corresponds to processes (6) - (8) in the above architecture diagram.
  * [**Authorization Using JWT Attributes**](/docs/authorize.md): Controls execution of APIs within the permitted range using attributes embedded in the token.
  * [**Sign-in with External IdP**](/docs/federation-signin.md): Adds sign-in with the tenant's custom IdP.

## Environment Deletion

* Delete the stack using the `cdk destroy` command.
* Delete the user pools created by the demo application.
* Delete the configuration information for the federated external IdP.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
