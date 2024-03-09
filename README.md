# TradingAppStoreExamples-C-Sharp
## Description
TradingApp.Store offers a comprehensive software suite enabling vendors to verify user permissions for their products. The suite comprises digitally signed Dynamic Link Libraries (DLLs) containing API functions accessible via C# scripts integrated into your software.

## Setup
Go to vendors.tradingapp.store, create a vendor account, and click the 'Create Listing' button at the top right. Proceed to fill out the Product Listing form.  Instructions below.

### Product Name:
Fill out a Product Name at the top left and a SKU will be automatically produced at the bottom of the page (this will identify your product on our servers). 

### Product Description:
Fill out a product description that fully explains your product, its benefits, and how it works.  Give as much detail as possible.  

### Listing Types:
Check all that apply to this product.  Your product may fit into more than one Listing Type.  

### Subscription Options:
Choose the type of billing scheme to use. (Such as Lifetime, Annual, Monthly, Free Trial + Monthly, etc)

### Upload Software Here:
***Skip this step temporarily because you first need to integrate the TAS subscription verification DLL into your custom software by following the directions below.  This will be the final step below before deployment.*

### Webhook Link:
If you have a real-time listening application that works with Webhooks, paste the link to it here to be notified when a purchase for this product is made.

### Purchase Email:
If you would like email notifications upon purchases, place the receiving address here.

### Target Platform:
Choose C_Sharp_App

### Platform Customer Data
If your software requires a username, you can put yours here. This value will be embedded into the license and can be checked at runtime. If no username, put "none".

### SKU:
Unique self-created product identifier that you will use in your script while accessing the DLL.

### Download MSI:
Press this to download a copy of the 'TradingApp.Store License Manager' that is master-keyed to this specific product and specifically to your Platform customer number.  This provides you, the Vendor, the ability to test the integrations of your products with our DLLs.  After installing this MSI, launch the TradingApp.Store license manager application to see the generated license.  


![TradingApp.Store License Manager](licensemanager_screenshot.png)  

Your system is now ready for seamless integration with our platform.



## How it works
The DLL will automatically detect a license in the TradingAppStore/licenses folder and then will determine if the user has permission. If the license is expired, or a newer version of the license is required, then the DLL will automatically update the license to contain the new information. Consequently, users need only execute the installer once to access any trading apps or software included with their current or future purchases.
The TradingAppStore DLL also offers a hardware authorization option that only allows a certain number of devices to access one instance of your product. This adds an additional layer of security by preventing copies of a DLL / product from gaining permission.
You may download the installer for TradingAppStore from the vendor portal whenever you are in the process of creating a listing. All licenses created from the vendor portal are tagged with a “Debug” flag, so they will not have any functionality in release mode. Thus, BE SURE TO CHANGE THE DEBUG FLAG TO FALSE AFTER COMPLETION OF TESTING PHASES.

## Implementation
Below is a C# implementation that calls the UserHasPermission function of the TradingAppStore DLL:
```C#
using System;
using static UserPermission;

class Program
{
    static void Main()
    {
        UserPermission p = new UserPermission();
        string productID =  "INSERT_PRODUCT_SKU";
        bool debug = true;

        //Perform user authentication using TAS authorization
        int error_machine_auth = p.GetMachineAuthorization(productID, debug);
        Console.WriteLine("Returned Error: " + error_machine_auth);

        if (error_machine_auth == 0)
            Console.WriteLine("Access granted");
        else
            Console.WriteLine("Access denied");
    }
}
```
To use the UserPermission namespace, you must add the TAS_DotNet.dll file located at C:\ProgramData\TradingAppStore\x64   (or x86) as a reference to your project.

## DLL Inputs
The DLL must have 2 input values:
* string productID :    SKU of the product to be checked.
* bool debug :          set to True if you are testing to use Debug licenses distributed by the vendor portal. SET TO FALSE FOR RELEASE OR ELSE ANYONE WILL HAVE ACCESS TO YOUR PRODUCT

## DLL Return Values
The DLL will return various error values based on numerous factors. It is up to your application how to handle them.
```
0 - no error
1 - expired
2 - wrong customerId
3 - cannot use Debug license in Release Mode
4 - invalid productId
5 - Too many user instances. Only for TAS Authorization. Contact support@tradingapp.store
6 - billing attempt not found... likely expired
8 - File Error
9 - other error
```

## Finishing Up
Go back to the Vendor Portal to complete your product setup.

### Upload Software Here:
Once your product is successfully integrated into our permissions system, take the product out of debug mode (see bool debug above), and export your project.  If you have accompanying files, workspaces, symbol lists, etc, zip everything into one file, and then upload it here.  This is what will be distributed to end-users at the time of purchase or free trial.

### Sales Information - Set Price:
This is the price per period for the subscription term of the product.  Revenue splits are explained in the Vendor Policy (https://tradingapp.store/pages/vendor-policy).

### Send for approval:
Click here to send this listing for approval by TAS site moderators.  You will be notified by email upon acceptance or rejection.


## Other Notes
If you are planning on using other apps sold from TradingApp.Store as an end-user, you must first uninstall the vendor installation and delete the TradingAppStore folder located at C:/ProgramData/ .  
This .bat will do that for you:  ![Delete TAS Folder Batch File](Delete_TAS_Folder.bat)
This will ensure that there will be no conflict between the license generated whenever you buy a real product and the debug license used for testing.

## Further Help
If you need assistance in implementation, you may email support@tradingapp.store and we will respond as quickly as possible.
