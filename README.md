# TradingAppStoreExamples-C-Sharp
## Description
TradingApp.Store offers a comprehensive software suite enabling vendors to verify user permissions for their products. The suite comprises digitally signed Dynamic Link Libraries (DLLs) containing API functions accessible via C# scripts integrated into your software.

## Setup
Go to [vendors.tradingapp.store](https://vendors.tradingapp.store/app), create a vendor account, and click the 'Create Listing' button at the top right. Proceed to fill out the Product Listing form.  Instructions below.

### Product Name:
Fill out a Product Name at the top left and a SKU will be automatically produced at the bottom of the page (this will identify your product on our servers). 

### Product Description:
Create a formated product description in an editor like Word or Google Docs and then paste it into the Product Description text-box. Read over it and do a final editing of the formatting to make sure it looks right. The product description should fully explain your product, its features, benefits, and how it works. Give as much detail as possible.

### Images:
Add images and/or videos one at a time. Most file common file types are accepted. You can also use online videos from YouTube via the URL link.

### Listing Types:
Check all that apply to this product.  Your product may fit into more than one Listing Type.  

### Subscription Options:
Choose the type of billing scheme to use. Available options are: Lifetime, Annual, Monthly, Free Trial + Monthly, Free Trial + Annual, Free Trial + Lifetime.

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
(NOTE: You must pick a 'Target Platform' and enter 'Platform Customer Data' in order for the 'Download MSI' button to appear.)
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
using System.Text;
using System.Text.Json;
class Program
{
    public static void Main(string[] args)
    {
        if (!VerifyDlls())
        {
            return; // VERY IMPORTANT: Handle the case for if verification fails. Do not use the library code! In this example, we simply return to terminate the program.
        }

        UserPermission p = new UserPermission();
        string productID = "INSERT_PRODUCT_SKU";
        bool debug = true; // VERY IMPORTANT: Only set this to true during testing. Actual implementation will have debug set to false.

        //Perform user authentication using TAS authorization
        int error_machine_auth = p.GetMachineAuthorization(productID, debug);
        Console.WriteLine("Returned Error: " + error_machine_auth);

        if (error_machine_auth == 0)
        {
            Console.WriteLine("Access granted");
        }
        else
        {
            Console.WriteLine("Access denied");
            return; // VERY IMPORTANT: Be sure to handle the case for when a user doesn't have access. In this example, we simply return to terminate the program.
        }
    }

    // Verifies our DLLs have not been tampered with.
    private static bool VerifyDlls()
    {
        // This gets a one-time-use magic number from our utility dll
        Utils utils = new Utils();
        string magicNumber = utils.ReceiveMagicNumber();
        
        // Now, let's send the magic number to our server for verificaiton
        var jsonString = JsonSerializer.Serialize(new { magic_number = magicNumber });
        using (var client = new HttpClient())
        {
            var content = new StringContent(jsonString, Encoding.UTF8, "application/json");
            var response = client.PostAsync("https://tradingstoreapi.ngrok.app/verifyDLL", content).Result;

            // Handle server respoonse accordingly
            if (response.StatusCode == System.Net.HttpStatusCode.OK)
            {
                Console.WriteLine("DLL accepted");
                return true;
            }
            else if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
            {
                Console.WriteLine("DLL has been tampered with.");
                return false;
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
                return false;
            }
        }
    }

}
```
To use the UserPermission namespace, you must add the TAS_DotNet.dll file located at C:\ProgramData\TradingAppStore\x64   (or x86, depending on your chosen architecture) as a reference to your project.

Similarly, to use the Utils namespace, add The Utils_DotNet file located at C:\ProgramData\TradingAppStore\x64   (or x86) as a reference to your project.

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
7 - internal error
8 - File Error
9 - other error
```

## Finishing Up
Go back to the Vendor Portal to complete your product setup.

### Upload Software Here:
Once your product is successfully integrated into our permissions system, take the product out of debug mode (see bool debug above), and create your deployment project.  If you have accompanying files, workspaces, symbol lists, etc, it is required that you zip everything into one file, and then upload it here.  This is what will be distributed to end-users at the time of purchase or free trial.

### Sales Information - Set Price:
This is the price per period for the subscription term of the product.  Revenue splits are explained in the Vendor Policy (https://tradingapp.store/pages/vendor-policy).

### Send for approval:
Click here to send this listing for approval by TAS site moderators.  You will be notified by email upon acceptance or rejection.


## Other Notes
If you are planning on using other apps sold from TradingApp.Store as an end-user, you must first uninstall the vendor installation of the TAS License Manager using Windows 'Add or Remove Programs' in Settings and then install the MSI received at the time of purchase.  We are currently working on building a dual-mode License Manager, so for now, you have to either work with two computers, or uninstall/reinstall if you want to switch from vendor-mode to end-user mode.

## Further Help
If you need assistance in implementation, you may email support@tradingapp.store and we will respond as quickly as possible.
