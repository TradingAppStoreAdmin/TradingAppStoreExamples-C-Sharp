# TradingAppStoreExamples-C-Sharp
## Description
TradingApp.Store offers a comprehensive software suite enabling vendors to verify user permissions for their products. The suite comprises digitally signed Dynamic Link Libraries (DLLs) containing API functions accessible via C# scripts integrated into your software.

## Setup
Go to [vendors.tradingapp.store](https://vendors.tradingapp.store/app), create or sign into your vendor account.  Should you already be an end-user of products in the trading app store, then you should create a vendor login using the same email address as your existing app store's end-user login.  These are two seperate systems, so you will need to register for each individually.  By using the same email address for each, the store will know to link your vendor account to your end-user account so that you will only have to install one License Manager.  Once your License Manager is installed, you will see a choice of flipping from 'Vendor Test Mode', back to 'End-User Mode' within the License Manager software.  After you created a new Vendor account, or logged in, then click the gear at the top right and choose 'Settings', then on the left side click 'Vendor Page' and setup your Vendor Profile page.  This is where you insert your company logo or icon (500x500 pixels works best.)  Next, enter a company bio or description, and contact information.  Now click 'Send for Approval'. Once approved this enters you into the  [Vendor Directory](https://tradingapp.store/pages/vendor-directory).  Next create your first product listing by clicking 'Listings' at the top of the page, and then 'Create Listing' button at the top right. Proceed to fill out the Product Listing form.  Instructions below.

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
Below is a C# implementation that calls the UserHasPermission function of the TradingAppStore DLL. Please review it carefully.
```C#
using System.Text;
using System.Text.Json;
using System.Reflection;
using System.Runtime.InteropServices;
class Program
{

    public static void Main(string[] args)
    {

        if (!VerifyDlls())
        {
            return; // VERY IMPORTANT: Handle the case for if verification fails. Do not use the library code! In this example, we simply return to terminate the program.
        }

        string productID = "INSERT_PRODUCT_SKU";
        bool debug = true; // VERY IMPORTANT: Only set this to true during testing. Actual implementation will have debug set to false.

        //Perform user authentication using TAS authorization
        int error_machine_auth = UseMachineAuthorization(productID, debug);
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

    // replace x64 with x86 if you are using 32 bit
    [DllImport("C:\\ProgramData\\TradingAppStore\\x64\\TASlicense.dll")]
    private static extern int UseMachineAuthorization(string productId, bool debug);
    
    // replace x64 with x86 if you are using 32 bit
    [DllImport("C:\\ProgramData\\TradingAppStore\\x64\\Utils.dll")]
    private static extern string GetMagicNumber();

    // Verifies our DLLs have not been tampered with.
    private static bool VerifyDlls()
    {
        // This gets a one-time-use magic number from our utility dll
        GetMagicNumber();
        string magicNumber = "";
        using (StreamReader s = new StreamReader("C:\\ProgramData\\TradingAppStore\\temp\\magic.txt"))
        {
            magicNumber = s.ReadToEnd();
        }

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
The TradingApp.Store License Manager has the ability to switch between ‘End-User’ mode, and ‘Vendor Test Mode’ by-way of a checkbox at the top right of the TAS Lisence Manager labeled:  ‘Vendor Test Mode’.
When it's checked, only Vendor's products show in License Manager’s DataGrids, when unchecked, only End-User products show in DataGrids.
If a Vendor is also an End-User of other’s products, they must use the same email address during registrations of  each mode for this to work.  This enables one installation of this License Manager to handle both integration testing while in Vendor Mode, and alternatively using End-User mode for purchased products use.


## Further Help
If you need assistance in implementation, you may email support@tradingapp.store and we will respond as quickly as possible.
