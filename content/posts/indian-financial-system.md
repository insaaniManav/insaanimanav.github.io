+++
title = "How Indian money actually moves"
date = "2026-10-09"
draft = false
author = "Manav"
+++

Let me set the scene for you, After 15 years, the Indian cricket team had stepped foot on Pakistani soil for an ODI series, Virendar Sehwag hits 309 and claims the title of "Multan ka Sultan" helping India win the ODI cup 3-2.

Several hundreds of kilometers away RBI had finally adopted a way to electronically transfer money in close to real time between 2 banks - Real-Time Gross Settlement - RTGS.

In the Early 2000s, India's economy was liberalising rapidly. The National Stock Exchange (NSE), the Bombay Stock Exchange (BSE), and the government bond markets were suddenly processing hundreds of billions of rupees daily.
The only way to move money around in these was high value cheques, It was a primitive system where an authenticated piece of paper had to move from your office or home to a depositor's bank, Where the following would happen

* By 11:00 AM the cheque had to reach the bank to be processed that day
* At 11:30 AM sharp the bank's courier would then have to rush to an RBI managed clearing house. Every major bank had a courier doing this exact same thing.
* At 12:00 PM the clearing house would finally process the amount manually by ensuring balances, authorization, authentication etc.
* At 1:00 PM The transaction would finally be input in a common ledger and sent to the RBI
* At 3:00 PM The transaction would be settled

Now you can imagine a founder sitting in Bangalore trying to pay 50 lakhs to a Mumbai based factory to start production would have to jump through a million hoops and ensure everything is in order to finally transfer this cash.

Apart from the inconvenience . What happens if Bank A has 10 people sending a total of 500cr via high value cheques which will be settled at 3pm and Bank A goes bankrupt at 2pm ?
Now Bank B owes Bank C 50cr and was dependent on this transaction and C further owes money. This is what the MBAs call "Settlement Risk", Something like this in a developing country could lead to a situation of total economic collapse [Ask the germans, they know](https://en.wikipedia.org/wiki/Herstatt_Bank)

Looking at all of this RBI finally decided to adopt and bring in the global standard of RTGS as a way to facilitate high value atomic transfers between entities.

## How does RTGS work

On a fundamental level RTGS is an atomic ledger update governs each bank's account with the RBI.

Similar to how we have an account with our bank, our bank has an account with the RBI. When you say give an account in bank B, 50 lakh rupees. RBI atomically deducts from A's account moves it to B's account and then B can settle it with the customer on their end.

In computer science, Atomic means "All or Nothing". That means the transaction has to either fully succeed or fully fail. If A's money is deducted but the network goes down. there have to be fail-safes in place to ensure A gets their money back and it appears that money was never deducted from A in the first place

In the real world, database atomicity translates into what central banks call Settlement Finality.

Because the database update is atomic, the exact moment the record commits on the central bank's ledger, the transaction becomes legally final, unconditional, and irrevocable. Even if Bank A goes completely bankrupt one minute later, Bank B legally owns that money, and the transaction cannot be unwound.
This strict technical atomicity is the main reason why all global financial systems rely on RTGS for high-value institutional payments, it completely eliminates the scenario of Settlement Risk.

---
Now that in 2004 high value low frequency transactions finally had a way to be processed, RBI wanted to tackle low value high frequency transactions too, AKA consumer retail. 

We just saw how RTGS settles every transaction individually against the RBI ledger, well the issue is, we can't do this a million times a day for 500 rupees each time. The biggest reason being every single settlement needs the sending bank to have that full amount sitting in its RBI account at that exact moment, and doing this a few times a day is acceptable but definitely not millions.

So in 2005 when people were queueing outside CD stores to buy Aashiq Banaya by Himesh Reshammiya, RBI introduced NEFT

## I've sent the money over, check in 30 minutes

NEFT was finally introduced as a way for customers to send small amounts of money to each other throughout the day except it wouldn't be immediate, it would be settled every 30 minutes.

The principle behind NEFT was similar to RTGS except instead of updating the ledger and settling bank accounts every few minutes, they would be all pushed into a queue and then after a fixed window a Delayed Net Settlement(DNS) would happen.

Let's say 10 people from bank A combined want to pay 5 lakhs to people of bank B and 15 people in B combined want to pay 15 lakhs to bank A combined, then in NEFT initially the banks would accumulate the transactions with themselves and post 30 minutes B would just settle and pay 10 lakhs to A.

No need to settle every small amount.

This meant that smaller transactions could be performed for super cheap and hence NEFT was offered for free for retail users.

This was also a massive liquidity relief because incoming and outgoing payments cancel each other out, commercial banks require drastically less raw fiat money in their RBI accounts to settle retail traffic.

## IFSC codes

When RTGS was first introduced, it was meant only for big corpos, branches of govt etc and wasn't really opened to the public. So whenever a big transfer happened from Bank A's account to Bank B, post the actual transfer, a clerk at bank B would manually lookup the branch of the account and transfer the money there.

But when NEFT came out, each transaction needed to land at the correct branch too post settlement, Asking a human to do it wasn't scalable. Hence Indian Financial System Code(IFSC) codes aka digital addresses for bank accounts were introduced.

**Wait but why isn't an account number enough ?**

As I am sure we have already kind of figured out, India's banking infrastructure evolved from thousands of completely isolated, legacy paper networks that were retrofitted into the digital era.
Hence, there is no universal authority that coordinates account numbers across the entire banking industry. Every bank is a private kingdom that designs its own internal database rules:

* Length Discrepancies: State Bank of India uses 11-digit account numbers, HDFC Bank uses 14 digits, and ICICI Bank uses 12 digits.
* The Overlap Nightmare: Because banks generate numbers independently, it is mathematically certain that Account Number 1234567890 exists simultaneously at SBI, HDFC, Axis, and Punjab National Bank.
* The Mainframe's Blindness: If you instruct the RBI's central NEFT mainframe to "Send ₹10,000 to Account 1234567890," the computer has no idea. It's like asking the postal service to send a letter to Room number 25. Which hotel, which locality where ??

Hence IFSC codes were born. This is what they look like in practice.

SBIN0000691 -  Bank code (4 chars) + Control Character (1 char - Always 0 for now) + Branch code(6 chars)

> Reminder to hydrate, It's okay just pick your nearest diet coke and take a sip

## NPCI, Wait another banking regulator ??

In 2008, RBI realized that the task of regulating the country's economy as a whole and managing millions of retail transactions a day was getting overwhelming and under the Payments and Settlements Act of 2007, NPCI was born as a non-profit org meant to take over all retail transactions.

One of the first things NPCI did was take control of something called a National Financial Switch NFS.

At its core NFS is an electronic junction box that connects the core banking mainframes of hundreds of different banks, allowing them to securely exchange transactional data packets in real time.

You know how you can just go to any ATM and withdraw cash instead of trying to find your bank's ATM, Yeah that was NFS. Initially for years, NFS was maintained and operated by RBI but NPCI took it over when it took over retail payments.

## IMPS - Instant payments for everyone at last

By 2010, millions of cheap mobile phones were flooding India, and the telecom sector was exploding. The RBI realized that the future of banking wasn't desktop internet banking (which NEFT relied on), it was mobile.

The NPCI hence designed a mobile-native architecture. They invented the MMID (Mobile Money Identifier), a simple 7-digit random number. This allowed a consumer to securely route an instant transaction using just a Mobile Number + MMID, completely bypassing the clunky IFSC structure.

#### But why couldn't we do this in NEFT era why DNS then ?

The RBI's central servers could not handle the processing stress of continuous, real-time atomic updates for millions of low-value consumer transactions.

The NPCI hence repurposed the high-speed NFS ATM switch they had just inherited.

That was it. Someone just figured out that this same comms layer on which every single ATM in the country already operated could do more than dispense cash.

Something that already had sub second latency and was secure from the ground up could also power the next wave of financial systems.

And it was already running 24/7/365. Bank holidays, middle of the night, it doesn't matter.

Hence IMPS was introduced, quick instant payments for everyone right on their smartphone.
It was as simple as -:

* Log on to your banking app
* Wait it didn't work, login again, Once more, lesgooo third time's the charm 
* Input OTP, wrong OTP, I didn't even type it I literally pasted it.
* Input Captcha, wrong captcha ?? Maybe I am a robot afterall
* Give up 
* Try after 10 minutes 
* Finally logged in
* Input some numbers
* Perfect how much you wanna pay 
* Payment failed, Oh wait but money got deducted
* They got the money too
* Payment failed 

## Birth of UPI

IMPS was finally a platform on top of which someone could do instant transactions between bank accounts for pennies. The issue was it still needed you to input a long code and then use a very well developed bank app.

UPI was finally introduced  the key to democratising digital transactions.

One VPA internally mapped to an account that could be in any bank, you could use any app to send or receive payments.

No restriction on the frontend, all the apps had to do was plug into NPCI's APIs through a partner bank and then they could put whatever they wanted before or after the payment.

Fun thing about VPAs they could just be converted into QR codes hung outside shops, stuck inside autos or probably the inside of a vadapav cover, pay when you finish eating.

Another interesting thing NPCI introduced with UPI was UPI collect, Finally a way for an app to send you a payment request instead of fumbling with net-banking or debit cards when you had to open amazon.

So the biggest reason IMO for UPI taking off was finally realizing customers really wanted to conveniently use their smartphones to pay and nobody liked their bank's horrible apps

## Conclusion

I feel like this is one of those places where India came out ahead, We built the rails as public infrastructure and let private companies fight over the interface with UPI apps.

Most countries did the opposite — card networks own the rails, charge 2-3%, and have no incentive to make transfers free.

So the next time you pay 50 rupees to an auto driver, Or when you pay 50 lakhs for a down payment, think about where the money comes from, where it goes to and how in mere seconds, Fiat Money, actual money moves bank accounts.