# 4 Refactoring Away from Exceptions

## Exceptions and Readability

dishonest functions => exception hide outcome of operation

goto = exception (even worse)

## Use Cases for Exceptions

always prefer using return values over exceptions

*   User Exceptions for exceptional situations
*   Exceptions should signalize a bug
*   Don't use exceptions in situation which you except to happen

## Fail Fast Principle

Stopping the current operation

Stateful => Process shutdown
Stateless => Operation shutdown (ASP.NET Core)

Advantages
* Shortening the feedback loop
* Confidence in the working software
* Protects the persistence state

## Where To Catch Exceptions

when to use:

* Generic Exception handler on the top of application => Error Handler in ASP.NET Core
* 3rd party library

## The Result Class

*   Help keep methods honest
*   Incorporates the of an operation with its status
*   Unified error model
*   Only for expected failures

Return Values to define an expected failure

```C#

    public class Result
    {
        public bool IsSuccess { get; }
        public Error Error { get; }
        public bool IsFailure => !IsSuccess;

        protected Result(bool isSuccess, Error error)
        {
            if (isSuccess && error != null)
                throw new InvalidOperationException();
            if (!isSuccess && error == null)
                throw new InvalidOperationException();

            IsSuccess = isSuccess;
            Error = error;
        }

        public static Result Fail(string message)
        {
            return new Result(false, message);
        }

        public static Result<T> Fail<T>(string message)
        {
            return new Result<T>(default(T), false, message);
        }

        public static Result Ok()
        {
            return new Result(true, string.Empty);
        }

        public static Result<T> Ok<T>(T value)
        {
            return new Result<T>(value, true, string.Empty);
        }

        public static Result Combine(params Result[] results)
        {
            foreach (Result result in results)
            {
                if (result.IsFailure)
                    return result;
            }

            return Ok();
        }
    }

    public class Result<T> : Result
    {
        private readonly T _value;
        public T Value
        {
            get
            {
                if (!IsSuccess)
                    throw new InvalidOperationException();

                return _value;
            }
        }

        protected internal Result(T value, bool isSuccess, string error)
            : base(isSuccess, error)
        {
            _value = value;
        }
    }

    public class Error : ValueObject
    {
        ErrorType  Type {get;}
        ErrorMessage Message {get;}
    }

```





