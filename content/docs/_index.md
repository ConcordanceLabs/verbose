---
title: Documentation
next: first-page
---

## Overview
Traditional application runtime environments operate with a single asynchronous event loop, like a timing belt that drives an application forward.

Blockchain-based applications and agent-to-agent systems has a unique application architecture, where federated environments, and their respective state move independent of other actors in the environment, yet the application requires these systems to work together closely to give the user an ideal experience. For this, a single asynchronous 'timing belt' alone is insufficient.

Rather than a single loop managing the application at runtime, we need an actuator; a mechanism to coordinate the progression of each independent environment without coupling them together.

<img width="1581" height="576" alt="image" src="https://github.com/user-attachments/assets/6e705b60-3dd7-45fe-9446-136f4f54db90" />



## Building Applications



{{< tabs >}}
{{< tab name="With Atara" >}}**With Atara**: Declarative code for error handling, timers, and listeners.  
``` Typescript {filename="tryAtara.ts", linenos=table}
export const waitforBuyer = (mintedOptionBody) => {
  client.on('buy', (id) => buyOption(id)).timeout(15000);
}

const buyer = waitForBuyer(mintedOption)
    .match(buyer)
       .with('success', () => exerciseOption(mintedOption) )
       .with('error', () => optionExpired(mintedOption));

```
{{< /tab >}}

  {{< tab name="Without Atara" >}}**Without Atara**: The same single function. Lower predictability. <br> Manually configured listeners and constraints like timers. Write rejections, awaits, error handling with nested conditional statements in your try catch verbosity makes applications brittle. 




``` Typescript {filename="withoutAtara.ts", linenos=table}
function timeout(ms: number) {
    return new Promise((_, reject) => 
        setTimeout(() => reject(new Error("BUYER_TIMEOUT")), ms)
    );
}

// A helper that waits for the purchase event
function waitForPurchase(vault: any, optionId: number, ms: number) {
    return new Promise((resolve, reject) => {
        const timer = setTimeout(() => {
            vault.off("OptionPurchased", listener); // Clean up listener on timeout
            reject(new Error("BUYER_TIMEOUT"));
        }, ms);

        const listener = (id: bigint) => {
            if (Number(id) === optionId) {
                clearTimeout(timer);
                vault.off("OptionPurchased", listener); // Clean up listener on success
                resolve(true);
            }
        };

        vault.on("OptionPurchased", listener);
    });
}

async function handleOptionLifecycle(vault: any, optionId: number) {
    try {
        // Race the event listener against a 15-second timer
        await waitForPurchase(vault, optionId, 15000);
        await vault.(optionId);

    } catch (error: any) {
        if (error.message === "BUYER_TIMEOUT") {
            console.log(`Timeout reached! No buyer for option #${optionId}. Cleaning up and returning collateral...`);
            
            await vault.cancelOrExpireUnboughtOption(optionId);   // Smart contract call to refund the seller

            
            console.log(`Option #${optionId} cleaned up successfully.`);
        } else {
            // Handle unexpected errors (e.g., RPC failure)
            console.error("An unexpected error occurred:", error);
        }
    }
}
```


.{{< /tab >}}
{{< /tabs >}}
