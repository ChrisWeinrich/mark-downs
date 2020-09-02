# 5 Creating an Anticorruption Layer

## Creating an Anticorruption Layer

Repositories are the Anticorruption Layer

```C#
 public class DeliveryRepository
    {
        public Delivery GetById(int id)
        {
            DeliveryLegacy legacyDelivery = GetLegacyDelivery(id);
            Delivery delivery = MapLegacyDelivery(legacyDelivery);

            return delivery;
        }

        private Delivery MapLegacyDelivery(DeliveryLegacy legacyDelivery)
        {
            if (legacyDelivery.CT_ST == null || !legacyDelivery.CT_ST.Contains(" "))
                throw new Exception("Invalid city and state");

            string[] cityAndState = legacyDelivery.CT_ST.Split(' ');

            var address = new Address(
                (legacyDelivery.STR ?? "").Trim(),
                cityAndState[0].Trim(),
                cityAndState[1].Trim(),
                (legacyDelivery.ZP ?? "").Trim());

            return new Delivery(legacyDelivery.NMB_CM, address);
        }

        private DeliveryLegacy GetLegacyDelivery(int id)
        {
            using (var connection = new SqlConnection(Settings.ConnectionString))
            {
                string query = @"
                    SELECT d.NMB_CLM, a.*
                    FROM [dbo].[DLVR_TBL] d
                    INNER JOIN [dbo].[ADDR_TBL] a ON a.DLVR = d.NMB_CLM
                    WHERE d.NMB_CLM = @ID";

                return connection
                    .Query<DeliveryLegacy>(query, new { ID = id })
                    .SingleOrDefault();
            }
        }

        private class DeliveryLegacy
        {
            public int NMB_CM { get; set; }
            public string STR { get; set; }
            public string CT_ST { get; set; }
            public string ZP { get; set; }
        }
    }
```

## Strengthening the Domain Model with Proper Encapsulation

Entity Base Class
Value Object Base Class

## Implementing the New Requirement

```C#
    public class ProductRepository
    {
        private const double PoundsInKilogram = 2.20462;

        public Product GetById(int id)
        {
            ProductLegacy legacyProduct = GetLegacyProduct(id);
            Product product = MapLegacyProduct(legacyProduct);

            return product;
        }

        private Product MapLegacyProduct(ProductLegacy legacyProduct)
        {
            if (legacyProduct.WT == null && legacyProduct.WT_KG == null)
                throw new Exception("Invalid weight in product: " + legacyProduct.NMB_CM);

            double weightInPounds = legacyProduct.WT ?? legacyProduct.WT_KG.Value * PoundsInKilogram;
            return new Product(legacyProduct.NMB_CM, weightInPounds);
        }

        private ProductLegacy GetLegacyProduct(int id)
        {
            using (var connection = new SqlConnection(Settings.ConnectionString))
            {
                string query = @"
                    SELECT NMB_CM, WT, WT_KG
                    FROM [dbo].[PRD_TBL]
                    WHERE NMB_CM = @ID";

                return connection
                    .Query<ProductLegacy>(query, new { ID = id })
                    .SingleOrDefault();
            }
        }

        private class ProductLegacy
        {
            public int NMB_CM { get; set; }
            public double? WT { get; set; }
            public double? WT_KG { get; set; }
        }
    }
```

```C#
    public class EstimateCalculator
    {
        private readonly DeliveryRepository _deliveryRepository;
        private readonly ProductRepository _productRepository;
        private readonly AddressResolver _addressResolver;

        public EstimateCalculator()
        {
            _deliveryRepository = new DeliveryRepository();
            _productRepository = new ProductRepository();
            _addressResolver = new AddressResolver();
        }

        public Result<decimal> Calculate(int deliveryId, int? productId1, int amount1,
            int? productId2, int amount2, int? productId3, int amount3, int? productId4, int amount4)
        {
            if (productId1 == null && productId2 == null && productId3 == null && productId4 == null)
                return Result.Fail<decimal>("Must provide at least 1 product");

            Delivery delivery = _deliveryRepository.GetById(deliveryId);
            if (delivery == null)
                throw new Exception("Delivery is not found for Id: " + deliveryId);

            double? distance = _addressResolver.GetDistanceTo(delivery.Address);
            if (distance == null)
                return Result.Fail<decimal>("Address is not found");

            List<ProductLine> productLines = new List<(int? productId, int amount)>
                {
                    (productId1, amount1),
                    (productId2, amount2),
                    (productId3, amount3),
                    (productId4, amount4)
                }
                .Where(x => x.productId != null)
                .Select(x => new ProductLine(_productRepository.GetById(x.productId.Value), x.amount))
                .ToList();

            if (productLines.Any(x => x.Product == null))
                throw new Exception("One of the products is not found");

            return Result.Ok(delivery.GetEstimate(distance.Value, productLines));
        }
    }

    public class AddressResolver
    {
        public double? GetDistanceTo(Address address)
        {
            /* Call to an external API */
            return 15;
        }
    }
```

## Validation Error vs preconditionial errors

**Exception** for Bugs (**Fault of Programmer**)  
**Validation Error** for wrong inputs (**Fault of User/Client**)

```C#
    public class Result
    {
        public bool IsSuccess { get; }
        public string Error { get; }
        public bool IsFailure => !IsSuccess;

        protected Result(bool isSuccess, string error)
        {
            if (isSuccess && error != string.Empty)
                throw new InvalidOperationException();
            if (!isSuccess && error == string.Empty)
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
```

Contract class for preconditions

```C#
  public static class Contracts
    {
        [DebuggerStepThrough]
        public static void Require(bool precondition, string message = "")
        {
            if (!precondition)
                throw new ContractException(message);
        }
    }

    [Serializable]
    public class ContractException : Exception
    {
        public ContractException()
        {
        }

        public ContractException(string message)
            : base(message)
        {
        }

        public ContractException(string message, Exception inner)
            : base(message, inner)
        {
        }

        protected ContractException(SerializationInfo info, StreamingContext context)
            : base(info, context)
        {
        }
    }
```