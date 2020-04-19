# 7 Handling Failures and Input Errors in a Functional Way

## Traditional Approach to Handling Failures and Input Errors

*   Handle all input errors at the boundaries of the domain model
*   Catch all expected failures at the lowest level possible

## Refactoring the Method Using the Result and Maybe Types

## Introducing Railway-oriented Programming

```C#
      public string RefillBalance(int customerId, decimal moneyAmount)
        {
            Result<MoneyToCharge> moneyToCharge = MoneyToCharge.Create(moneyAmount);
            Result<Customer> customer = _database.GetById(customerId).ToResult("Customer is not found");

            return Result.Combine(moneyToCharge, customer)
                .OnSuccess(() => customer.Value.AddBalance(moneyToCharge.Value))
                .OnSuccess(() => _paymentGateway.ChargePayment(customer.Value.BillingInfo, moneyToCharge.Value))
                .OnSuccess(
                    () => _database.Save(customer.Value)
                        .OnFailure(() => _paymentGateway.RollbackLastTransaction()))
                .OnBoth(result => Log(result))
                .OnBoth(result => result.IsSuccess ? "OK" : result.Error);
        }

```

```C#
    public static class ResultExtensions
    {
        public static Result<T> ToResult<T>(this Maybe<T> maybe, string errorMessage) where T : class
        {
            if (maybe.HasNoValue)
                return Result.Fail<T>(errorMessage);

            return Result.Ok(maybe.Value);
        }

        public static Result OnSuccess(this Result result, Action action)
        {
            if (result.IsFailure)
                return result;

            action();

            return Result.Ok();
        }

        public static Result OnSuccess(this Result result, Func<Result> func)
        {
            if (result.IsFailure)
                return result;
            
            return func();
        }

        public static Result OnFailure(this Result result, Action action)
        {
            if (result.IsFailure)
            {
                action();
            }

            return result;
        }

        public static Result OnBoth(this Result result, Action<Result> action)
        {
            action(result);

            return result;
        }

        public static T OnBoth<T>(this Result result, Func<Result, T> func)
        {
            return func(result);
        }
    }
```