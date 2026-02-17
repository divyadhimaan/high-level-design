# Payment Gateway

> Any system used to settle financial transactions through the transfer of monetary values.

## Step 1: Understanding Design Scope

- Payment system can be many things
  - digital wallet like google pay
  - backend system handling payments like stripe and paypal

> Q: What kind of payment system are we building?
>
> A: A payment backend for an e-commerce system, similar to Amazon.com. It handles everything related to money movement.

> Q: What payment options are supported - Credit cards, PayPal, bank cards, etc?
>
> A: The system should support all these options in real life. For the purposes of the interview, we can use credit card payments.

> Q: Do we handle credit card processing ourselves?
>
> A: No, we use a third-party provider like Stripe, Braintree, Square, etc.

> Q: Do we store credit card data in our system?
> 
> A: Due to compliance reasons, we do not store credit card data directly in our systems. We rely on third-party payment processors.

> Q: Is the application global? Do we need to support different currencies and international payments?
> 
> A: The application is global, but we assume only one currency is used for the purposes of the interview.

> Q: How many payment transactions per day do we support?
>
> A: 1mil transactions per day.

> Q: Do we need to support the payout flow to eg payout to payers each month?
> 
> A: Yes, we need to support that

> Q: Is there anything else I should pay attention to?
> 
> A: We need to support reconciliations to fix any inconsistencies in communicating with internal and external systems.