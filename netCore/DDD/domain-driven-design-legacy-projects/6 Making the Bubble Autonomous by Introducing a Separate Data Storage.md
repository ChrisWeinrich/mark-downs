# 6 Making the Bubble Autonomous by Introducing a Separate Data Storage

## Synchronizing the Anticorruption Layer

Problem => Data is only in legacy bounded Context

Synchronize old with new Database

## Creating a new Database

## Adjusting the Domain Model and Persistence Logic

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


```

## Recap: Creating a New Database

*  New Database own format

## Identifying a New Entry Point for the Bubble




