# Week2 Group Work

<details>
<summary><h2>Original Work</h2></summary>

<details>
  <summary>SubscriptionServiceTest</summary>
  
``` java
package com.kenzie.subscribeandsave.service;

import com.kenzie.subscribeandsave.App;
import com.kenzie.subscribeandsave.types.Subscription;
import com.kenzie.subscribeandsave.util.SubscriptionRestorer;
import org.apache.commons.lang.StringUtils;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.fail;

public class SubscriptionServiceTest {
    private static final String ASIN = "B01BMDAVIY";
    private static final String CUSTOMER_ID = "amzn1.account.AEZI3A0633629PUR8GGGS9ZSC3DO";
    private static final String SUBSCRIPTION_ID = "81a9792e-9b4c-4090-aac8-28e733ac2f54";

    private SubscriptionService classUnderTest;

    /**
     * The entry point, which results in calls to all test methods.
     *
     * @param args Command line arguments (ignored).
     */
    public static void main(String[] args) {
        SubscriptionServiceTest tester = new SubscriptionServiceTest();

        // clean up subscriptions before/after running runAllTests(), for when tests are invoked via main(),
        // rather than through a build
        tester.restoreSubscriptions();
        tester.runAllTests();
        tester.restoreSubscriptions();
    }

    @Test
    public void runAllTests() {
        classUnderTest = new SubscriptionService(App.getAmazonIdentityService(), App.getSubscriptionDAO(),
                                                 App.getAmazonProductService());
        boolean pass = true;

        pass = subscribe_newSubscription_subscriptionReturned();
        pass = getSubscription_existingSubscription_subscriptionReturned() && pass;
        pass = subscribe_unknownCustomer_exceptionOccurs() && pass;
        pass = test_bug1() && pass;
        pass = test_bug2() && pass;
        pass = test_bug3() && pass;

        if (!pass) {
            String errorMessage = "\n/!\\ /!\\ /!\\ The SubscriptionService tests failed. Test aborted. /!\\ /!\\ /!\\";
            System.out.println(errorMessage);
            fail(errorMessage);
        } else {
            System.out.println("The SubscriptionService tests passed!");
        }
    }

    public boolean getSubscription_existingSubscription_subscriptionReturned() {
        // GIVEN - a valid subscriptionId
        String subscriptionId = SUBSCRIPTION_ID;

        // WHEN - get the corresponding subscription
        Subscription result = classUnderTest.getSubscription(subscriptionId);

        // THEN - a subscription object is returned, with a matching id
        if (result == null) {
            System.out.println("   FAIL: Getting a subscription for a valid if should return the subscription.");
            return false;
        }
        if (!subscriptionId.equals(result.getId())) {
            System.out.println("   FAIL: Subscription returned when getting subscription by id has mismatching id " +
                                   "value");
            return false;
        }

        System.out.println("  PASS: Getting a subscription for a valid id succeeded.");
        return true;
    }

    public boolean subscribe_newSubscription_subscriptionReturned() {
        // GIVEN - a customerId to make a subscription for, asin to subscribe to, and the frequency to receive
        // subscription
        String customerId = CUSTOMER_ID;
        String asin = ASIN;
        int frequency = 1;

        // WHEN - create a new subscription
        Subscription result = classUnderTest.subscribe(customerId, asin, frequency);

        // THEN a subscription should be returned and the id field should be populated
        if (result == null) {
            System.out.println("   FAIL: Creating subscription should return the subscription.");
            return false;
        }
        if (StringUtils.isBlank(result.getId())) {
            System.out.println("   FAIL: Creating subscription should return a subscription with a populated id " +
                                   "field.");
            return false;
        }

        System.out.println("  PASS: Creating a new subscription succeeded.");
        return true;
    }

    public boolean subscribe_unknownCustomer_exceptionOccurs() {
        // GIVEN - an invalid customerId to make a subscription for
        String customerId = "12345678";
        String asin = ASIN;
        int frequency = 1;

        // WHEN/THEN - try to create a new subscription, catch IllegalArgumentException
        try {
            Subscription result = classUnderTest.subscribe(customerId, asin, frequency);
        } catch (IllegalArgumentException w) {
            System.out.println("  PASS: Cannot subscribe with invalid customerId.");
            return true;
        }

        System.out.println("   FAIL: An exception should have occurred when subscribing invalid customer.");
        return false;
    }

    // PARTICIPANTS: Fill in the example test below after fixing Bug 1 - refactor as needed
    public boolean test_bug1() {
        // GIVEN

        // WHEN

        // THEN

        System.out.println("   FAIL: Need to implement test to fix Bug 1!");
        return false;
    }

    // PARTICIPANTS: Rename and fill in the example test below after fixing Bug 2 - refactor as needed
    public boolean test_bug2() {
        // GIVEN

        // WHEN

        // THEN

        System.out.println("   FAIL: Need to implement test to fix Bug 2!");
        return false;
    }

    // PARTICIPANTS: Rename and fill in the example test below after fixing Bug 3 - refactor as needed
    public boolean test_bug3() {
        // GIVEN

        // WHEN

        // THEN

        System.out.println("   FAIL: Need to implement test to fix Bug 3!");
        return false;
    }


    @BeforeEach
    @AfterEach
    public void restoreSubscriptions() {
        SubscriptionRestorer.restoreSubscriptions();
    }
}
```
</details>

<details>
  <summary>SubscriptionFileStorage</summary>
  
``` java
package com.kenzie.subscribeandsave.dao;

import com.kenzie.subscribeandsave.types.Subscription;

import org.apache.commons.io.FileUtils;

import java.io.File;
import java.io.IOException;
import java.nio.charset.Charset;
import java.util.List;
import java.util.UUID;

/**
 * Subscription data store that is file based.
 */
public class SubscriptionFileStorage {

    private File subscriptionsFile;

    /**
     * Creates a {@code SubscriptionFileStorage} using the specified file for reading/writing subscriptions.
     *
     * @param subscriptionsFile The subscription {@code File} to use
     */
    public SubscriptionFileStorage(File subscriptionsFile) {
        this.subscriptionsFile = subscriptionsFile;
    }

    /**
     * Creates a new subscription.
     * <p>
     * Throws {@code StorageException} if the subscription already exists or if an input/output error occurs.
     *
     * @param subscription the subscription to store
     * @return The subscription that was written
     */
    public Subscription writeSubscription(Subscription subscription) {
        Subscription existingSubscription = getSubscription(subscription.getCustomerId(), subscription.getAsin());

        if (existingSubscription != null) {
            throw new StorageException(
                    String.format("Subscription already exists: %s. The update API is coming soon! " +
                            "Please cut a ticket for this to be done manually.", existingSubscription));
        }

        String id = UUID.randomUUID().toString();
        subscription.setId(id);


        StringBuilder sb = new StringBuilder();
        sb.append(subscription.getId()).append(",");
        sb.append(subscription.getCustomerId()).append(",");
        sb.append(subscription.getAsin()).append(",");
        sb.append(subscription.getFrequency());
        sb.append("\n");

        try {
            FileUtils.writeStringToFile(subscriptionsFile, sb.toString(), Charset.defaultCharset(), true);
        } catch (IOException e) {
            throw new StorageException("Unable to save subscription.", e);
        }

        return subscription;
    }

    /**
     * Updates an existing subscription.
     * <p>
     * Throws {@code IllegalArgumentException} if the {@code Subscription} is null, missing an ID or if no
     * subscription is found for that ID.
     * <p>
     * Throws {@code StorageException} if an error occurs trying to write the updated record.
     *
     * @param subscriptionId The {@code Subscription} to update (must already have a subscription ID)
     * @return the {@code Subscription} if writing succeeded
     */
    public Subscription getSubscriptionById(String subscriptionId) {
        String[] lines = readSubscriptionsFile();
        for (String subscriptionLine : lines) {
            String[] subscriptionData = subscriptionLine.split(",");
            String id = subscriptionData[0].trim();
            if (subscriptionId.equals(id)) {
                Subscription subscription = Subscription.builder()
                        .withSubscriptionId(id)
                        .withAsin(subscriptionData[1])
                        .withCustomerId(subscriptionData[2])
                        .withFrequency(Integer.parseInt(subscriptionData[3]))
                        .build();
                return subscription;
            }
        }

        return null;
    }

    private Subscription getSubscription(String customerId, String asin) {
        String[] lines = readSubscriptionsFile();
        for (String subscriptionLine : lines) {
            String[] subscriptionData = subscriptionLine.split(",");

            String currentCustomerId = subscriptionData[1].trim();
            String currentAsin = subscriptionData[2].trim();

            if (customerId.equals(currentCustomerId) && asin.equals(currentAsin)) {
                Subscription subscription = Subscription.builder()
                        .withSubscriptionId(subscriptionData[0].trim())
                        .withAsin(currentAsin)
                        .withCustomerId(currentCustomerId)
                        .withFrequency(Integer.parseInt(subscriptionData[3].trim()))
                        .build();
                return subscription;
            }
        }

        return null;
    }

    private String[] readSubscriptionsFile() {
        try {
            List<String> lines = FileUtils.readLines(subscriptionsFile, Charset.defaultCharset());
            return lines.toArray(new String[lines.size()]);
        } catch (IOException e) {
            throw new StorageException("Unable to access subscription data.", e);
        }
    }
}
```
</details>

<details>
  <summary>SubscriptionService</summary>
  
``` java
package com.kenzie.subscribeandsave.service;

import com.kenzie.subscribeandsave.dao.SubscriptionDAO;
import com.kenzie.subscribeandsave.resources.AmazonIdentityService;
import com.kenzie.subscribeandsave.resources.AmazonProductService;
import com.kenzie.subscribeandsave.resources.Product;
import com.kenzie.subscribeandsave.types.Subscription;

import org.apache.commons.lang3.StringUtils;

/**
 * Subscribe and service API. Currently supports creating subscriptions and fetching them. Subscriptions can
 * only be made for valid amazon products by valid amazon customers. Subscriptions are persisted by the SubscriptionDAO.
 */
public class SubscriptionService {

    private AmazonIdentityService identityService;
    private AmazonProductService productService;
    private SubscriptionDAO subscriptionDAO;

    /**
     * Creates new subscription service instance with the given dependencies.
     *
     * @param identityService The identity service to use for validating customers
     * @param subscriptionDAO The subscription DAO for reading/writing subscriptions
     * @param productService  The product service to use for validating/getting products
     */
    public SubscriptionService(AmazonIdentityService identityService,
                               SubscriptionDAO subscriptionDAO,
                               AmazonProductService productService) {
        this.identityService = identityService;
        this.subscriptionDAO = subscriptionDAO;
        this.productService = productService;
    }

    /**
     * Creates a new subscription for given customer and ASIN for the given frequency (given in months between
     * deliveries).
     * <p>
     * Throws {@code IllegalArgumentException} if customer ID is blank/invalid, ASIN is blank/invalid, ASIN
     * is unsubscribable, or if frequency is invalid (less than 1 or greater than 6).
     *
     * @param customerId The customer's ID
     * @param asin       The ASIN of the product to subscribe customer to
     * @param frequency  The frequency of delivery (delivery every N months)
     * @return the new {@code Subscription} if successful, {@code null} otherwise
     */
    public Subscription subscribe(String customerId, String asin, int frequency) {
        if (StringUtils.isBlank(customerId) || StringUtils.isBlank(asin)) {
            throw new IllegalArgumentException(
                String.format(
                    "Invalid inputs. A Customer ID and ASIN must be provided. Provided: {Customer ID: %s, ASIN: %s}",
                    customerId,
                    asin)
            );
        }

        if (frequency < 1 || frequency > 6) {
            throw new IllegalArgumentException(
                String.format(
                    "Invalid frequency value. Please provide how often (in months) the " +
                        "subscription should occur - between 1 and 6. Provided: {Frequency: %d}",
                    frequency));
        }

        if (!identityService.validateCustomer(customerId)) {
            throw new IllegalArgumentException(
                String.format("Unable to create subscription for customerId: %s. Unknown customer.", customerId)
            );
        }

        Product product = productService.getProductByAsin(asin);
        if (product == null) {
            throw new IllegalArgumentException(
                String.format("Unable to create subscription for ASIN: % s. Unrecognized ASIN.", asin)
            );
        }

        return subscriptionDAO.createSubscription(customerId, asin, frequency);
    }

    /**
     * Returns the {@code Subscription} corresponding to the given subscription ID.
     *
     * @param subscriptionId The ID of the subscription to fetch
     * @return the {@code Subscription} if one is found, {@code null} otherwise
     */
    public Subscription getSubscription(String subscriptionId) {
        if (StringUtils.isBlank(subscriptionId)) {
            throw new IllegalArgumentException("A subscriptionId must be provided.");
        }

        return subscriptionDAO.getSubscription(subscriptionId);
    }
}
```
</details>

</details>
