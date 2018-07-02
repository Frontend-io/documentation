# Test Cards

Here’s are some cards that work on our test environment.
## Working Cards
### No validation

```
Card Number: 408 408 408 408 408 1
Expiry Date: any date in the future
CVV: 408
```

### PIN+OTP validation
*(nonreusable)*

```
Card Number: 5060 6666 6666 6666 666 (Verve)
Expiry Date: any date in the future
CVV: 123
PIN: 1234
OTP: 123456
```

### PIN only validation
*(reusable)*

```
Card Number: 5078 5078 5078 5078 12 (Verve)
Expiry Date: any date in the future
CVV: 081
PIN: 1111
```

### PIN+PHONE+OTP validation

```
Card Number: 5078 5078 5078 5078 4 (Verve)
Expiry Date: any date in the future
CVV: 884
PIN: 0000
Phone: If less than 10 numeric characters, Transaction will fail.
OTP: 123456
```

### Bank authorization Simulation

```
Card Number: 408 408 0000000 409
Expiry Date: any date in the future
CVV: 000
```

## Cards failing with error message
### Insufficient Balance
```
Make any payment where amount > N500,000
```

### Declined
```
Card Number: 408 408 0000005 408
Expiry Date: any date in the future
CVV: 001
```

### Token Not Generated. Customer Not Registered on Token Platform
```
Card Number: 5078 5078 5078 5078 53 (Verve)
Expiry Date: any date in the future
CVV: 082
PIN: (any PIN)
```

### 500 error
```
Card Number: 506066 0000000064 (Verve)
Expiry Date: any date in the future
CVV: 606
```

### Timeout error
```
Card Number: 506066 506066 506067 (Verve)
Expiry Date: any date in the future
CVV: 060
```