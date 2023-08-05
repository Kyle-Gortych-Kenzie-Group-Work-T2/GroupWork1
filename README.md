# GroupWork1

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

    /*
    public boolean subscribe_invalidFrequency_throwsIllegalArgumentException() {
        //Given
        String customerId = CUSTOMER_ID;
        String validAsin = ASIN;
        int invalidFrequency = 7;
        IllegalArgumentException exception = null;
        // WHEN
        try {

        }
    }
     */


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
  <summary>Bug_1_Test_Case</summary>
  
``` java
/*add*/
```
</details>

<details>
  <summary>Bug_2_Test_Case</summary>
  
``` java
/*add*/
```
</details>

<details>
  <summary>Bug_3_Test_Case</summary>
  
``` java
/*add*/
```
</details>
