<!--
# license: Copyright © 2011-2016 Razer Merchant Services Sdn Bhd. All Rights Reserved. 
-->

# rms-mobile-xdk-xamarin-ios

<img src="https://user-images.githubusercontent.com/38641542/74424311-a9d64000-4e8c-11ea-8d80-d811cfe66972.jpg">

This is the complete and functional Razer Merchant Services Xamarin iOS payment module that is ready to be implemented into Xamarin iOS project through C# file copy and paste. An example application project (MOLPayXDKExample) is provided for MOLPayXDK Xamarin iOS integration reference.

This plugin provides an integrated Razer Merchant Services payment module that contains a wrapper 'MOLPay.cs' and an upgradable core as the 'molpay-mobile-xdk-www' folder, which the latter can be separately downloaded at https://github.com/RazerMS/rms-mobile-xdk-www and update the local version.

## Recommended configurations

    - Xamarin Studio 6.0 ++

    - Package Json.NET

    - Minimum iOS target version: 8.0

## Installation

    Step 1 - Copy and paste MOLPay.cs into the project folder of your Xamarin iOS project. Right click on the project name in the Solution of Xamarin Studio, go to Add -> Add Files..., select MOLPay.cs and click Open.

    Step 2 - Copy and paste molpay-mobile-xdk-www folder (can be separately downloaded at https://github.com/RazerMS/rms-mobile-xdk-www) into the Content\ folder of your Xamarin iOS project. Right click on the Resources\ folder in the Solution of Xamarin Studio, go to Add -> Add Existing Folder..., select molpay-mobile-xdk-www folder and click Open. 
    
    Step 3 - Copy and paste custom.css into the Resources\ folder of your Xamarin iOS project. Right click on the Resources\ folder in the Solution of Xamarin Studio, go to Add -> Add Files..., select custom.css and click Open.   
    
    Step 4 - Right click on every added file in the Solution of Xamarin Studio, go to Build Action and make sure BundleResource is checked.

    Step 5 - Open Info.plist and add this key value pair of type String "NSPhotoLibraryUsageDescription" : "Payment images".

    Step 6 - Open Info.plist and add this key value pair of type String "NSPhotoLibraryAddUsageDescription" : "Payment images".

    Step 7 - Add package Json.NET by going to Project -> Add NuGet Packages..., check Json.NET and click Add Package.

    Step 8 - Add the result callback function.
    
    public void MolpayCallback(string transactionResult)
    {
        Console.WriteLine("MolpayCallback transactionResult = " + transactionResult);
        NavigationController.PopViewController(false);
    }
    
    =========================================
    Sample transaction result in JSON string:
    =========================================
    
    {"status_code":"11","amount":"1.01","chksum":"34a9ec11a5b79f31a15176ffbcac76cd","pInstruction":0,"msgType":"C6","paydate":1459240430,"order_id":"3q3rux7dj","err_desc":"","channel":"Credit","app_code":"439187","txn_ID":"6936766"}
    
    Parameter and meaning:
    
    "status_code" - "00" for Success, "11" for Failed, "22" for *Pending. 
    (*Pending status only applicable to cash channels only)
    "amount" - The transaction amount
    "paydate" - The transaction date
    "order_id" - The transaction order id
    "channel" - The transaction channel description
    "txn_ID" - The transaction id generated by Razer Merchant Services
    
    * Notes: You may ignore other parameters and values not stated above
    
    =====================================
    * Sample error result in JSON string:
    =====================================
    
    {"Error":"Communication Error"}
    
    Parameter and meaning:
    
    "Communication Error" - Error starting a payment process due to several possible reasons, please contact Razer Merchant Services support should the error persists.
    1) Internet not available.
    2) API credentials (username, password, merchant id, verify key).
    3) Razer Merchant Services server offline.

## Import namespaces

    using System.Collections.Generic;
    using System.IO;
    using Foundation;
    using Newtonsoft.Json;
    using MOLPayXDK;

## Prepare the Payment detail object

    Dictionary<String, object> paymentDetails = new Dictionary<String, object>();

    // Optional, REQUIRED when use online Sandbox environment and account credentials.
    paymentDetails.Add(MOLPay.mp_dev_mode, false);

    // Mandatory String. Values obtained from MOLPay
    paymentDetails.Add(MOLPay.mp_username, "");
    paymentDetails.Add(MOLPay.mp_password, "");
    paymentDetails.Add(MOLPay.mp_merchant_ID, "");
    paymentDetails.Add(MOLPay.mp_app_name, "");
    paymentDetails.Add(MOLPay.mp_verification_key, "");

    // Mandatory String. Payment values
    paymentDetails.Add(MOLPay.mp_amount, ""); // Minimum 1.01
    paymentDetails.Add(MOLPay.mp_order_ID, "");
    paymentDetails.Add(MOLPay.mp_currency, "");
    paymentDetails.Add(MOLPay.mp_country, "");

    // Optional, but required payment values. User input will be required when values not passed.
    paymentDetails.Add(MOLPay.mp_channel, ""); // Use 'multi' for all available channels option. For individual channel seletion, please refer to https://github.com/RazerMS/rms-mobile-xdk-examples/blob/master/channel_list.tsv. 
    paymentDetails.Add(MOLPay.mp_bill_description, "");
    paymentDetails.Add(MOLPay.mp_bill_name, "");
    paymentDetails.Add(MOLPay.mp_bill_email, "");
    paymentDetails.Add(MOLPay.mp_bill_mobile, "");

    // Optional, allow channel selection. 
    paymentDetails.Add(MOLPay.mp_channel_editing, false);

    // Optional, allow billing information editing.    
    paymentDetails.Add(MOLPay.mp_editing_enabled, false);

    // Optional for Escrow
    paymentDetails.Add(MOLPay.mp_is_escrow, ""); // Optional for Escrow, put "1" to enable escrow

    // Optional, for credit card BIN restrictions and campaigns.
    String[] binlock = new String[] { "", "" };
    paymentDetails.Add(MOLPay.mp_bin_lock, binlock);

    // Optional, for mp_bin_lock alert error.
    paymentDetails.Add(MOLPay.mp_bin_lock_err_msg, "");

    // WARNING! FOR TRANSACTION QUERY USE ONLY, DO NOT USE THIS ON PAYMENT PROCESS.
    // Optional, provide a valid cash channel transaction id here will display a payment instruction screen. Required if mp_request_type is 'Receipt'.
    paymentDetails.Add(MOLPay.mp_transaction_id, "");
    // Optional, use 'Receipt' for Cash channels, and 'Status' for transaction status query.
    paymentDetails.Add(MOLPay.mp_request_type, "");

    // Optional, use this to customize the UI theme for the payment info screen, the original XDK custom.css file can be obtained at https://github.com/RazerMS/rms-mobile-xdk-examples/blob/master/custom.css.
    paymentDetails.Add(MOLPay.mp_custom_css_url, Path.Combine(NSBundle.MainBundle.BundlePath, "Content/custom.css"));

    // Optional, set the token id to nominate a preferred token as the default selection, set "new" to allow new card only.
    paymentDetails.Add(MOLPay.mp_preferred_token, "");

    // Optional, credit card transaction type, set "AUTH" to authorize the transaction.
    paymentDetails.Add(MOLPay.mp_tcctype, "");

    // Optional, required valid credit card channel, set true to process this transaction through the recurring api, please refer the MOLPay Recurring API pdf 
    paymentDetails.Add(MOLPay.mp_is_recurring, false);

    // Optional, show nominated channels.
    String[] allowedChannels = new String[] { "", "" };
    paymentDetails.Add(MOLPay.mp_allowed_channels, allowedChannels);

    // Optional, simulate offline payment, set boolean value to enable. 
    paymentDetails.Add(MOLPay.mp_sandbox_mode, false);

    // Optional, required a valid mp_channel value, this will skip the payment info page and go direct to the payment screen.
    paymentDetails.Add(MOLPay.mp_express_mode, false);

    // Optional, extended email format validation based on W3C standards.
    paymentDetails.Add(MOLPay.mp_advanced_email_validation_enabled, false);

    // Optional, extended phone format validation based on Google i18n standards.
    paymentDetails.Add(MOLPay.mp_advanced_phone_validation_enabled, false);

    // Optional, explicitly force disable user input.
    paymentDetails.Add(MOLPay.mp_bill_name_edit_disabled, true);
    paymentDetails.Add(MOLPay.mp_bill_email_edit_disabled, true);
    paymentDetails.Add(MOLPay.mp_bill_mobile_edit_disabled, true);
    paymentDetails.Add(MOLPay.mp_bill_description_edit_disabled, true);

    // Optional, EN, MS, VI, TH, FIL, MY, KM, ID, ZH.
    paymentDetails.Add(MOLPay.mp_language, "EN");

    // Optional, Cash channel payment request expiration duration in hour.
    //paymentDetails.Add(MOLPay.mp_cash_waittime, "48");

    // Optional, allow non-3ds on some credit card channels.
    //paymentDetails.Add(MOLPay.mp_non_3DS, false);

    // Optional, disable card list option.
    paymentDetails.Add(MOLPay.mp_card_list_disabled, false);

    // Optional for channels restriction, this option has less priority than mp_allowed_channels.
    String disabledChannels[] = {"credit"};
    paymentDetails.Add(MOLPay.mp_disabled_channels, disabledChannels);


## Start the payment module UI

    MOLPay molpay = new MOLPay(paymentDetails, MolpayCallback);
    molpay.Title = "MOLPayXDK";
    NavigationController.PushViewController(molpay, false);

## Cash channel payment process (How does it work?)

    This is how the cash channels work on XDK:
    
    1) The user initiate a cash payment, upon completed, the XDK will pause at the “Payment instruction” screen, the results would return a pending status.
    
    2) The user can then click on “Close” to exit the Razer Merchant Services XDK aka the payment screen.
    
    3) When later in time, the user would arrive at say 7-Eleven to make the payment, the host app then can call the XDK again to display the “Payment Instruction” again, then it has to pass in all the payment details like it will for the standard payment process, only this time, the host app will have to also pass in an extra value in the payment details, it’s the “mp_transaction_id”, the value has to be the same transaction returned in the results from the XDK earlier during the completion of the transaction. If the transaction id provided is accurate, the XDK will instead show the “Payment Instruction" in place of the standard payment screen.
    
    4) After the user done the paying at the 7-Eleven counter, they can close and exit Razer Merchant Services XDK by clicking the “Close” button again.

## XDK built-in checksum validator caveats 

    All XDK come with a built-in checksum validator to validate all incoming checksums and return the validation result through the "mp_secured_verified" parameter. However, this mechanism will fail and always return false if merchants are implementing the private secret key (which the latter is highly recommended and prefereable.) If you would choose to implement the private secret key, you may ignore the "mp_secured_verified" and send the checksum back to your server for validation. 

## Private Secret Key checksum validation formula

    chksum = MD5(mp_merchant_ID + results.msgType + results.txn_ID + results.amount + results.status_code + merchant_private_secret_key)

## Support

Submit issue to this repository or email to our support-sa@razer.com

Merchant Technical Support / Customer Care : support-sa@razer.com<br>
Sales/Reseller Enquiry : sales-sa@razer.com<br>
Marketing Campaign : marketing-sa@razer.com<br>
Channel/Partner Enquiry : channel-sa@razer.com<br>
Media Contact : media-sa@razer.com<br>
R&D and Tech-related Suggestion : technical-sa@razer.com<br>
Abuse Reporting : abuse-sa@razer.com
