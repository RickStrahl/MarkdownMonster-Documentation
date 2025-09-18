Markdown Monster can be downloaded and evaluated for free, but a license <a href="https://markdownmonster.west-wind.com/pricing.aspx" target="top">must be purchased for continued use</a>. Licenses are per user, rather than per machine, so a licensed user can use Markdown Monster on a limited number of computers as needed. Various license packs and an organizational license are also available to use Markdown Monster for multiple installations within a single organization. 

Licenses are valid for the major version for which it was purchased (ie. v3.0 to v3.99).

For pricing and purchasing of a license please check this link:

* [Markdown Monster Pricing and Purchases](https://markdownmonster.west-wind.com/pricing.aspx)

### Registering 
Purchases provide you with a Registration Key that you can apply to a Markdown Monster installation. To register your instance of Markdown Monster:

* Open Markdown Monster
* Go to **Help -> Software Registration**
* Enter your Registration Key into the text box

Here's what the registration form looks like:

![Registration Form](/images/RegistrationForm.png)

Once registered the **(Unregistered)** header disappears from the toolbar and all of the registration reminder dialog popups no longer pop up.

> #### @icon-info-circle Each Machine Requires Registration
> If you install Markdown Monster on multiple machines, each machine requires its own registration - registrations are never shared across machines due to the machine specific registration key **even if you share configuration settings**. 

### Registration Checks
Due to the high volume of fraudulent license attempts Markdown Monster now validates licenses occasionally. In order to validate a license **an Internet Connection is required**. Licenses are checked once a week, and if not online are flagged for removal if the next online license check fails as well. 

Once a license is removed Markdown Monster reverts back to **Unregistered** mode with registration reminders and some advanced features disabled. To restore the license, you have to re-enter your license information in the registration form.

> We realize that this may cause an occasional un-registration of a valid license, but with registration rates running close to 1-100 with registered vs. fake registrations we needed a mechanism to disallow bypassing the registration check mechanism. 

### Key Storage
When Markdown Monster is registered it registers the current instance with a machine specific key that is stored on your local hard drive as `Registered.key`.

* In the Markdown Monster Local App Data Folder (`%localappdata%\Markdown Monster`)

### @icon-warning-color:indianred Long Time User: Registration lost after Update?
If you're a long time user of Markdown Monster, you may run into a problem with an existing registration not being recognized after an update. 

The reason for this is a change in how Markdown Monster is installed: We moved from the original install location in `%localappData%\Markdown Monster` to `Program Files\Markdown Monster`, and the install folder is now completely deleted and reinstalled on updates. Versions that were installed long ago default to the `%localappdata%` folder, and since the license key is stored there, the key gets wiped out along with rest of the application on every update.
 
#### To fix this Issue: Uninstall and Reinstall!
Uninstall Markdown Monster explicitly (from Windows **Add or remove Programs**) and then reinstall. This re-installs into `Program Files` and properly puts the license key in the `%localappdata%` folder that is no longer cleared.

Note that you can change the install folder to any location you like, **except** the `%localappdata%\Markdown Monster` folder.