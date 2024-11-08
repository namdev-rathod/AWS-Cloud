### **AWS WAF Captcha Implementation - Using Nodejs Lambda Function & AWS CLI** 🚀

---

### Using AWS CLI

1. Install required AWS CLI latest version
2. Configure IAM credentials and region 
3. Run below command on command prompt to create AWS WAF Captcha API Key

```
aws wafv2 create-api-key --scope REGIONAL --token-domains web.awsguruji.net

```

---

### Using Nodejs Lambda Function

1. Login to AWS Console
2. Navigate to Lambda function service
3. Create new lambda function using nodejs 18
4. Copy below function code and execute it
5. Use attached zip aws sdk node modules for supported lambda layers

 - IAM Role with WAF Permission For Lambda Function
 
```

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "wafv2:CreateAPIKey",
                "wafv2:ListAPIKeys",
                "wafv2:DeleteAPIKey"
            ],
            "Resource": "*"
        }
    ]
}

```

- Lambda Function To Create WAF Captcha API Key

```

const AWS = require('aws-sdk');
const wafv2 = new AWS.WAFV2();

exports.handler = async (event) => {
    const scope = 'REGIONAL'; // Use 'REGIONAL' for API Gateway or ALB, 'CLOUDFRONT' for CloudFront
    const tokenDomains = ['dev.awsguruji.net']; // List of domains using the CAPTCHA API Key

    try {
        // Call the createAPIKey method with TokenDomains
        const result = await wafv2.createAPIKey({
            Scope: scope,
            TokenDomains: tokenDomains,
        }).promise();

        // Extract the API Key and API Key ID
        const apiKey = result.APIKey.APIKey;
        const apiKeyId = result.APIKey.APIKeyId;
        
        console.log(`API Key: ${apiKey}`);
        console.log(`API Key ID: ${apiKeyId}`);

        // Response format for Lambda
        return {
            statusCode: 200,
            body: JSON.stringify({
                message: 'WAF CAPTCHA API Key created successfully',
                apiKey: apiKey,
                apiKeyId: apiKeyId,
            }),
        };

    } catch (error) {
        console.error('Error creating WAF API Key:', error);

        // Handle errors in Lambda response format
        return {
            statusCode: 500,
            body: JSON.stringify({
                message: 'Failed to create WAF CAPTCHA API Key',
                error: error.message,
            }),
        };
    }
};

```


---

### Ref

- https://docs.aws.amazon.com/cli/latest/reference/wafv2/create-api-key.html
- https://aws.amazon.com/blogs/security/use-aws-waf-captcha-to-protect-your-application-against-common-bot-traffic/
- https://docs.aws.amazon.com/waf/latest/developerguide/waf-captcha-and-challenge.html

📢 **Subscribe to My YouTube Channel!** 📢 


If you’re excited about mastering Terraform from beginner to expert, 
make sure to **hit the Subscribe button**  [DevOps With Namdev](https://www.youtube.com/@namdev.devops)!


By subscribing, you’ll get access to the latest tutorials, real-world projects, and deep dives designed to help you master DevOps skills from the ground up. 
Hit that **Subscribe** button to stay updated and take your expertise to the next level! 🎬👨‍💻