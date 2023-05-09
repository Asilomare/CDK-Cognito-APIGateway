# CDK-Cognito-API

This application seamlessly integrates AWS services such as API Gateway, Cognito, Boto3, and Lambda to establish a robust and efficient API. The API facilitates seamless provisioning and interaction with a Cognito backend. By leveraging these powerful AWS services, the application ensures a scalable, secure, and high-performance solution for user authentication and management.

## Methods

Signup and Auth methods use boto3 to interact with the Cognito UserPool

### Signup

This endpoint, accessible at awesome-domain.com/auth, is designed to initiate the creation of a new user.
The Cognito stack contains commented-out custom configurations of password requirements, multi-factor authentication (MFA), and custom onboarding or authentication Lambda functions for you to tailor to your needs.

Example Usage
```
import requests
import json

link = 'https://.example.com/{}' # or 'https://vl01lja6v5.execute-api.us-east-1.amazonaws.com/prod/{}'

def signup(username, temporary_password, email):
   
    headers = {'Content-Type': 'application/json'}
    
    payload = {
        'username': username,
        'email': email,
        'temporary_password': temporary_password
    }
    
    res = requests.post(link.format('signup'), params=payload, headers=headers)
    
    return json.loads(res.text)
```

### Auth

The endpoint located at awesome-domain.com/auth is designed to handle user authentication and guide them through any necessary additional steps, in line with AWS Cognito standards. Upon successful authentication or initiation of a challenge, such as email verification or password reset, the function returns a session token that is valid for up to one day.

AWS Cognito follows a secure and flexible authentication flow, allowing developers to implement multiple authentication steps, such as MFA or custom challenge verification. With this endpoint, users can easily navigate the authentication process while ensuring a high level of security and compliance with AWS Cognito best practices.

Example Usage:
```
import requests
import json

link = 'https://.example.com/{}' # or 'https://vl01lja6v5.execute-api.us-east-1.amazonaws.com/prod/{}'

def auth(username, password):
    
    headers = {'Content-Type': 'application/json'}
    
    payload = {
        'username': username,
        'password': password
    }
    
    res = requests.post(link.format('login'), params=payload, headers=headers)
    
    return json.loads(res.text)
```

### Challenge

This endpoint is meant to handle challenges returned by the auth endpoint. Reachable at '/challenge', this endpoint is used in conjunction with '/auth' endpoint.

Example Usage:
```
import requests
import json

link = 'https://.example.com/{}' # or 'https://vl01lja6v5.execute-api.us-east-1.amazonaws.com/prod/{}'

def challenge(username, temporary_password, new_password):
    
    auth_data = auth(username, temporary_password)
    
    if 'error' in auth_data: return auth_data
    
    challenge_name = auth_data['ChallengeName']
    session = auth_data['Session']
    
    headers = {'Content-Type': 'application/json'}
    
    payload = {
        'challenge_name': challenge_name,
        'session': session,
        'challenge_responses': json.dumps({
            'USERNAME': username,
            'NEW_PASSWORD': new_password
        })
    }
    
    res = requests.post(link.format('challenge'), params=payload, headers=headers)
    
    return json.loads(res.text)
```

### GPT

This method showcases how to secure an endpoint using JWT cookies provided by cognito in conjunction with authorizers.
First obtain a JWT token 

Example Usage:
```
import requests
import json

link = 'https://.example.com/{}' # or 'https://vl01lja6v5.execute-api.us-east-1.amazonaws.com/prod/{}'

def authenticated_request(username, password):
    payload = auth(username, password)
    
    auth_token = payload['id_token']
    
    api_key = '<OPENAI-KEY>'
    message = '<GPT PROMPT>'
    
    headers = {
        'Auth': auth_token,
        'Content-Type': 'application/json'
    }
    
    payload = {
        'key': api_key,
        "message": message
    }
    
    res = requests.post(link.format('gpt'), params=payload, headers=headers)
    
    return json.loads(res.text)
```

## Configuration

Domain Name: To configure the domain name, navigate to stacks/api_stack.py and locate the definition of the 'api' variable. A valid SSL/TLS certificate for the website, stored in AWS Certificate Manager, is required for this configuration.

Authentication Options: During instantiation, you can customize the authentication process to fit your requirements. This includes defining custom onboarding/authentication Lambdas, enabling Multi-Factor Authentication (MFA), customizing SMS messages, and setting password requirements. Fine-grained control over the authentication flow allows for a tailored and secure user experience.
