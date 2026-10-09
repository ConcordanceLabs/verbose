---
title: Documentation
next: first-page
---

This is a demo.

## Building Applications



{{< tabs >}}
{{< tab name="Tacit" >}}**With Tacit**: Declarative code that handles errors and listeners.  
``` Typescript {filename="Tecit.ts", linenos=table}
export const waitforBuyer = () => {
  client.on('buy', (id) => buyOption(id)).timeout(15000);
}

const buyer = waitforBuyer()
    .match(buyer)
       .with('success', () => exerciseOption() )
       .with('error', () => optionExpired());

```
{{< /tab >}}

  {{< tab name="Without Tacit" >}}**Without Tacit**: The same single function. Manually configured your listeners and constraints like timers. Rejections, awaits and error handling with conditional statements inside try/catch verbosity makes application predictability harder. 




``` Typescript {filename="Without.ts", linenos=table}
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
