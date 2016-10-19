##Integrating CloverGo SDK 

###First Steps

Add the **CloverGo.framework**

Other Dependent Libraries that needs to linked to your iOS Application

- Foundation.framework
- CoreBluetooth.framework
- ExternalAccessory.framework
- MobileCoreServices.framework
- SystemConfiguration.framework
- MediaPlayer.framework
- AudioToolbox.framework
- AVFoundation.framework
- libstdc++.6.tbd

###Additional project setting

In Build Settings under **Target** and set **Allow Non-modular Includes in Framework Modules** to **_YES_**.

###Initializing the SDK

```
- (id)initWithEmployeeId:(NSString *)employeeId
              merchantID:(NSString *)merchantId
                deviceId:(NSString *)deviceId
```

Initialization should be done only once and you can get the CloverGo instance rest of the application using 

```
[CloverGo sharedInstance]
```

###Before Initializing the Card Reader

The view controller that initializes the card reader should implement **_CloverGoCardReaderDelegate_**

Below are the methods that should be implemented by the View Controller that acts as the **_CloverGoCardReaderDelegate_**

```
- (void)onReaderConnected;

- (void)onReaderReady;

- (void)onReaderDisconnected;

- (void)onReaderResetProgress:(CloverGoReaderResetProgress*)progress;

- (void)onSelectAidFromList:(NSArray *)arrayOfAID
                 completion:(void (^)(CloverGoEMVApplicationIdentifier* aid))completion;

- (void)onCardReadProgress:(CloverGoCardReaderEvent*)event;

- (void)onCardReaderError:(CloverGoCardReaderEvent*)event;

- (void)onSelectReaderFromList:(NSArray *)arrayOfReaders
                 completion:(void (^)(CloverGoCardReaderInfo* cardReader))completion;

@optional
- (void)onTransactionAbort;
```

Set the cardReaderDelegate variable in the cloverGoInstance with the ViewController that implements the **_CloverGoCardReaderDelegate_**

```
_cloverGoInstance.cardReaderDelegate = self;
```

###Initialize the Card Reader
```
[[CloverGo sharedInstance] initCardReader:CloverGoCardReaderType450 shouldAutoReset:YES]
```

CloverGo will scan for Bluetooth readers available and the send the list will sent to the callback method **_onSelectReaderFromList_** in the **_CloverGoCardReaderDelegate_**. 
App needs to choose the reader it needs to connect with start the initializing process. **_CloverGoCardReaderDelegate_** will provide status updates during the different stages of initialization

###Before Initializing a Transaction

The View Controller that handles the payment transaction processing should implement **_CloverGoTransactionDelegate_**

Below are the methods that should be implemented by the View Controller that acts as the **_CloverGoTransactionDelegate_**

```
- (void)onSuccess:(CloverGoTransactionResponse*)response;

- (void)onFailure:(CloverGoTransactionError*)error;

- (void)proceedOnError:(CloverGoTransactionEvent*)error
            completion:(void (^)(BOOL proceed))completion;
```

Set the transactionDelegate variable in the cloverGoInstance with the View Controller that implements the **_CloverGoTransactionDelegate_**

```
_cloverGoInstance.transactionDelegate = self;
```

###Initiate a Transaction

Once the reader is initialized, the status of the reader can be verified using **_isCardReaderReady_** method

```
[[CloverGo sharedInstance] isCardReaderReady]
```

Create the **_CloverGoTransactionRequest_** with the following method, please note all amounts are in pennies

```
+ (instancetype) requestWithAmount:(NSDecimalNumber*)subTotal
                               tax:(NSDecimalNumber*)tax
                               tip:(NSDecimalNumber*)tip
                 externalPaymentId:(NSString*)externalPaymentId
                         orderItems:(NSArray*)orderItems;
```

Call the **_doTransaction_** method in CloverGo instance to initiate the transaction

```
[[CloverGo sharedInstance] doTransaction:request];
```

Once the transaction is complete, the response **_CloverGoTransactionResponse_** will be sent to the **_CloverGoTransactionDelegate_** method **_onSuccess_**

###Additional Transactional properties

Additional transactional properties can be set on the CloverGo instance to 
- Allow Duplicate Transactions
- Ignore AVS Check

```
[[CloverGo sharedInstance] allowDuplicateTransactions:YES];
[[CloverGo sharedInstance] ignoreAVSCheck:YES];
```

###Get Inventory

The View Controller that handles the inventory items should implement **_CloverGoInventoryDelegate_**

Below are the methods that should be implemented by the View Controller that acts as the **_CloverGoInventoryDelegate_**

```
-(void) getInventoryItemsSuccess:(NSArray*) inventoryItems;

-(void) getInventoryItemsFailure:(NSError*) error;
```

To get an array of **_CloverGoInventoryItem_** call the **_getInventoryItems_** method

```
[[CloverGo sharedInstance] getInventoryItems];
```

###Get Tax Rates

The View Controller that handles the Tax Rates should implement **_CloverGoTaxRateDelegate_**

Below are the methods that should be implemented by the View Controller that acts as the **_CloverGoTaxRateDelegate_**

```
-(void) getTaxRatesSuccess:(NSArray*) taxRates;

-(void) getTaxRatesFailure:(NSError*) error;
```

To get an array of **_CloverGoTaxRate_** call the **_getTaxRates_** method

```
[[CloverGo sharedInstance] getTaxRates];
```

###Enabling Debug Mode

To enable CloverGo SDK’s debug logs to be printed on the console, call the **_setDebugMode_** method with a **_YES_** Boolean parameter

```
[[CloverGo sharedInstance] setDebugMode:YES];
```

