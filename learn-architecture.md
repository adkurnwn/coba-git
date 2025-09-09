Diagrams: __[Draw.io](https://drive.google.com/file/d/1hz0jkLynZEvWcFjUCIe3CR1SdUr0T3Lu/view?usp=sharing)__

### 1. Microservices Architecture
![Microservices](https://i.imgur.com/A3LbX7v_d.webp?maxwidth=1520&fidelity=grand)

Microservices are used in large, fast-changing systems where scalability, independent deployment, and team flexibility are crucial. commonly used in e-commerce, fintech, streaming, and SaaS platforms. Being used to deploy changes for a specific service, without the threat of bringing down the entire application. Commonly used tech stack: `Golang, Python + FastAPI/Flask/Django, and NodeJS`.

Microservices architecture has API Gateway. With API Gateway, routes incoming requests to the right service, Handles authentication, rate limiting, and logging. Commonly used API Gateway tech stack: `Kong, NGINX`.

In microservices, each service typically manages its own database (to avoid tight coupling)



### 2. Monolitics Architecture
![Monolitics](https://i.imgur.com/Bhk9quf_d.webp?maxwidth=1520&fidelity=grand)

Monolithic architecture is used in small to medium systems where simplicity, tight integration, and centralized management are more important than flexibility. Commonly used in system with less user, such as internal business tools, traditional enterprise apps.

Being used to develop, deploy, and run the entire application as a single unit, which makes it easier to build and maintain initially but harder to scale or update individual features without affecting the whole system. If there’s an error in any module, it could affect the entire application’s availability