# Flash-Modding-Guide
In-depth ActionScript + JPEXS Decompiler guide for Flash Reverse Engineering.
### What will we do?

In this guide, we'll dive deep into the world of Flash reverse engineering with a focus on modding Stick Arena Private Servers. We'll start by decompiling the Stick Arena SWF file to understand its underlying ActionScript code. Through a series of step-by-step tutorials, we'll modify game mechanics, add new features, and even introduce custom game elements. By the end of this guide, you'll not only have a customized version of Stick Arena but also a solid foundation in Flash reverse engineering techniques that can be applied to other projects.

### Prerequisites:
* Install Andre-Jar's emulator and set up locally: https://github.com/andre-jar/Stick-Arena-Private-Server
* A Flash decompiler, recommended JPEXS
* Understand what decompilation and pseudo-code is
* SWF file (this guide uses Stick Arena 558, soon to be updated to 588)
* At least one of a Flash Emulator such as Ruffle, A Flash-compatible Browser such as FlashBrowser, or an archived Flash Player application. Recommend having all three.

### Nice to have:
* Basic understanding of ActionScript code
* Basic understanding of MySQL
* VScode or equivalent text editor (at least something better than notepad)
* Understanding of tools for debugging such as a browser with a working Network Tab such as Pale Moon (I haven't used this yet)
* NGROK for HTTP request interceptions (will be covered in a later version, may not be necessary if you work in a Flash-compatible browser with a working Network tab)
* Patience

  ---

# Guides

### Changing Color Palette [easy]
- prereqs: JPEXS 

Step 1: Navigate: /sprites/DefineSprite (5215)

Step 2: Expand DefineSprite (5215) folder, expand frame 1. Identify "PlaceObjectX (XXXX)"

Step 3: Navigate to /shapes/DefineShape(XXXX) manually or click "XXXX" under "Needed Characters"

Step 4: Right click DefineSprite (XXXX) and click replace. Upload image of color palette, can use a PNG from google by searching "color palette" on google images.

![image](https://github.com/Ebubekir-Tas/Flash-Modding-Guide/assets/65694925/3811d893-b53a-453b-ad1f-ee18ea224ae3)

### Changing Character's Head Color [medium]
- prereqs: JPEXS, text editor for SVG editing

Step 1: Navigate to character's head sprite /sprites/DefineSprite (974) or (1007) in 588.

Step 2: Expand frame 1. Identify "PlaceObject2 (7914)

Step 3: Navigate to /shapes/DefineShapeX (7914)

Step 4: Right click and export selection, export as SVG.

Step 5: Navigate to SVG file and open in text editor. Find the HEX code, for example #000000 and replace with new hex code such as #0066CC

Step 6: Go back to the same DefineShape (7914) file, right click and Replace

Step: 7 Replace with the SVG that you modified the hex color of.

optional repeat steps with sprite 301.

I made mine Spongebob, this requires making your own custom SVG.

![image](https://github.com/Ebubekir-Tas/Flash-Modding-Guide/assets/65694925/7c04296f-6b64-475b-8642-31de968ac829)


### Extend spinner shop with custom spinners [expert]
- prereqs: Everything. Don't try this at home.

This is a long one so instead of being split into steps it will be split into parts. 

Pt 1.
-

Recommended: Export /scripts folder to open in text editor such as vscode for better searching and code parsing.

Identify `maxShopSpinners` variable. I think it's initially 41. Around line 8424 of the ActionScript code. This number is responsible for the length of the spinner shop. In order to add more spinners that can be purchased this number must be extended. Any unused places will result in an `undefined` slot in the shop, so recommend making this variable the actual number of spinner placements necessary rather than an arbitrarily high number such as 99.

Recommended: Understand how the shop works. 
`/scripts/frame5.DoAction.as`:

At line 154, `function initializeSpinnersShop()` this holds some of the behavior of the shop.

code:
```
   spinnersMC.gotoAndStop("shop");
   var i = 0;
   while(i < maxShopSpinners)
   { //... }
```

This creates slots in the shop for a spinner to be purchased that correspond to the value of maxShopSpinners. If the number is 100, it will create 100 slots to buy spinners in the store (slots meaning when you click the left and right arrow button to find a spinner. For example Stealth I believe is slot 35.)

code: `eval("spinnersMC.subSpinnersMC.indicatorMC" + selectedShopSpinner).play();`

indicatorMC corresponds to the item ID of the value. An example of a spinner would be "indicatorMC35" which would correspond to the Stealth spinner. This is how the store links your purchase in the shop to the spinner itself. You don't have to worry about subIndicator.

Pt 2.
-

In frame5 file:

```
function validatePurchaseClientSide(purchaseCost)
{
   if(Number(player.zqkPmT) >= purchaseCost)
   {
      shopPurchaseCost = purchaseCost;
      displayShopMessage("\r" + shopText[20] + purchaseCost.toString() + shopText[21],"window304ShopConfirmMessageMC",true);
      return true;
   }
   displayShopMessage(shopText[31],"window304ShopNotEnoughCredMessageMC",true);
   return false;
}
```

This determines if you have enough creds. This is important because the spinner in your shop may appear to be purchased client sided but not actually be purchased, it will show the purchase message *if you can afford the spinner when purchasing*, not if the spinner has actually been applied to your account.

```
function performShopPurchase()
{
   if(shopPurchaseType == _shopSpinnerType)
   {
      player.spinners[player.spinners.length] = {num:selectedShopSpinner,red0:cR[0],green0:cG[0],blue0:cB[0],red1:cR[1],green1:cG[1],blue1:cB[1]};
      player.sendBuyItem(100 + selectedShopSpinner,cR[0],cG[0],cB[0],cR[1],cG[1],cB[1]);
   }
   else if(shopPurchaseType == _shopPetType)
   {
      player.pets[player.pets.length] = {num:selectedShopPet + 1,red0:cR[0],green0:cG[0],blue0:cB[0],red1:cR[1],green1:cG[1],blue1:cB[1]};
      player.sendBuyItem(200 + selectedShopPet + 1,cR[0],cG[0],cB[0],cR[1],cG[1],cB[1]);
   }
   else if(shopPurchaseType == _shopMapSlotType)
   {
      player.jhJkpo = Number(player.jhJkpo) + 1;
      updateMapSlotsShop();
      player.sendBuyItem(240,0,0,0,0,0,0);
   }
   player.zqkPmT = Number(player.zqkPmT) - shopPurchaseCost;
   credText0.text = credText1.text = formatNumber(player.zqkPmT);
   displayShopMessage(purchaseSuccessMessage,"window304ShopMessageMC",true);
}
```

Spinner and Pet purchases, as well as map purchases are all handled through the same function. It checks if your purchase type is 100 (buy spinners), 200 (buy pet), or type 240 (purchase map)

This is not an actionable step, however it's important to understand what's happening 

Pt 3.
-

```
function updateLocalPlayerSpinnerAndPet()
{
   var _loc2_ = player.spinners[player.jhHtWQ];
   fqTEcOL.attachMovie("indicatorMC" + padStr(_loc2_.num,2),"charIndicatorMC",1555);
   setCustomObject2Colors(fqTEcOL.charIndicatorMC.subIndicatorMC.colorMC0,fqTEcOL.charIndicatorMC.subIndicatorMC.colorMC1,_loc2_.red0,_loc2_.green0,_loc2_.blue0,_loc2_.red1,_loc2_.green1,_loc2_.blue1,75);
   var _loc1_ = player.pets[player.vwEGyv];
   gxyAWm.attachMovie("petBehavior" + petBehavior[Number(_loc1_.num)] + "MC","charPetMC",7777);
   var _loc3_ = gxyAWm.charPetMC.subPetMC.attachMovie("petMC" + padStr(_loc1_.num,2),"petMC",1,{_x:70,_y:-80});
   _loc3_._xscale = _loc3_._yscale = petScale;
   setCustomObject2Colors(_loc3_.colorMC0,_loc3_.colorMC1,_loc1_.red0,_loc1_.green0,_loc1_.blue0,_loc1_.red1,_loc1_.green1,_loc1_.blue1,100,true);
}
```

This is a good example of how pets and spinners work.

`fqTEcOL.attachMovie("indicatorMC" + padStr(_loc2_.num,2),"charIndicatorMC",1555);`

All spinners have a preface of indicatorMC followed by their item ID. padStr(_loc2_.num,2) is basically the item ID that's being concatenated to the indicatorMC string. So if that's 35, the result will be "indicatorMC35". Everything after that including "charIndicatorMC" and "1555" are other arguments to the attachMovie function that's not relevant here.

Pt 4.
-

Search for spinners by searching JPEXS with CTRL + F and searching keyword of `indicatorMC` because remember all spinners have the prefix, followed by the item ID. So if we know the Builder spinner is ID of 81, the Builder sprite will be named `DefineSprite (XXXX: indicatorMC81)` for example. Replacing existing spinners can be a different guide, here we want to extend the shop with new spinners.

You have to do a bit thinking here if using a different swf version, but assuming you're using version 558 the steps should be the same as this guide. You have to find the item ID (indicator ID) of the last spinner that can be bought in the shop. You can do this by scrolling all the way to the right in the shop (assuming you didn't change the `maxShopSpin`), and buy that last spinner. Then you can check the database for that item ID. In my case the last item ID I think was 51, I don't know for sure but let's say it was 51. That means my new custom spinner should be item ID of 52. Keep in mind there are some exclusive spinners with higher numbered item ID's, for example Moderator spinner is ID 83, Builder 81, etc. To make these purchasable in the shop you have to lower the ID's to correspond to the slot in the shop. Those steps have not included yet, but let's make it a step now since we will have to go over it anyway. So this will be a sub-guide of how to make the Mod and Builder spinners purchasable. This will bring us mostly to our goal of adding new spinners as this would cover how to add item ID's to the shop.

Pt 5.
-

So let's add the Moderator spinner to our shop. The first thing we need to do is extend the length of our shop. Navigate to `scripts/frame1.DoAction.as` and find var `maxShopSpinners`. For me it is on line 8424. Change this number from whatever it is, plus one. So if it is 41 change it to 42. If you change it to 43 and navigate to the 43rd slot, it will show an undefined slot. This is why we want to change the Moderator spinner from 83 to 43, because otherwise you will have to scroll through 40 undefined slots in order to reach the spinner. But of course if you don't want the Moderator spinner purchasable then I guess later you can change it back to 83 but don't be lazy because it's important to know how to change the indicatorMC number from 83 to your required number in this case. We have to change it in JPEXS. Remember to save your progress.

For this purpose I'm going to choose an arbitrary number of 55. We will change the Moderator spinner to 55 because I've already edited my shop a bit and it can really work with any number, if you choose to also use 55 you'll just have to scroll through undefined slots a couple times. It doesn't matter what number we choose in reality since this is temporary.

next part: Changing Moderator spinner (83) to slot 55 

Pt 6.
-

Navigate Moderator Spinner in JPEX. CTRL + F for `indicatorMC83` in JPEXS.
DO NOT click on `DefineSprite 7911`. Like, ever. It'll crash JPEXS.
Expand `DefineSprite (6803: indicatorMC83)` as it is defined in my case. Expand frame 1.
Click on the `PlaceObject` inside frame 1. We've already learned how to replace frames in previous guides such as changing the Color Palette. This is one step further. If you click the `PlaceObject` you will see on the bottom left of JPEXS the tag type, in my case `PlaceObject2`, and Character Id of 6802. We won't touch this right now, but this is to learn what Character Id and Tag Type is.
Click on the Define Sprite of the Moderator sprite. In my case it is `Define Sprite(6803: indicatorMC83)`. Take a minute to think to yourself before proceeding and answer the question: What do we have to change from this in order to put the Moderator spinner in our shop? Read the previous parts of this guide. Answer: the number after `indicatorMC` corresponds to the slots number inside the shop, and `maxShopSpinners` determines how many slots there are. So if `indicatorMC` is 55 and our `maxShopSpinners` variable is set to 55, then the Moderator spinner will be the last slot in our shop for purchase. If you haven't by now change your `maxShopSpinner` variable to 55. So then we have to change `indicatorMC` number, right? From `indicatorMC83` to `indicatorMC55`. But when we look at the bottom left of JPEXS, we see Character Id of 6803 and Tag Type of Define Sprite. And if we right click Define Sprite, frame 1, PlaceObject, we don't see anything to change that number. There are things you can edit but do not bother. You won't find where to update indicatorMC83 to indicatorMC55 by clicking in this area. You must follow the next step.


Pt. 7
-

In order to change the number 83 from `indicatorMC83` we must first determine where that 83 number comes from.

Search in JPEXS `indicatorMC83`. Be sure you are in `resources` in your JPEXS in the File tab. Navigate to `others/ExportAssets (6803: indicatorMC3)`.

This export is what determines the number of the spinner. This is the missing link, the holy grail. In SAPS there are a lot of red herrings, and one of those red herrings was thinking it was possible to change the number of `indicatorMC83` within that same folder, through terminal commands or something else. If you change ExportAssets the DefineSprite will change as well, they are linked with the ExportAssets tag being the source of truth.

BE CAREFUL HERE. Take a screenshot of your `ExportAsset`, back it up, whatever. I recommend you install a backup sab558.swf anyway.

Click on `ExportAssets (6849: indicatorMC83)`. in JPEXS  click "edit" on the bottom. You will see assets with `forceWriteAsLong ...` underneath. click on the + on the left of the text `assets`. You will see: `tag[0]: U16 = 6849, name[0]: String = indicatorMC83`. Double Click on `indicatorMC83` (be sure you clicked edit in JPEXS) and change the text to `indicatorMC55` and click the save button on the bottom. Give JPEXS some seconds to process the change, and then click save AGAIN on the top left on JPEX, the floppy disk in the File tab.

Note that now the Moderator spinner in the sprite folder should reflect the label with `indicatorMC55`.

Now make sure in `scripts/frame1/DoAction.as` you have `maxShopSpinners` set to 55, maybe set to 56 or 60 just in case so you see what happens when the number is too high.

And when that's all saved, SAVE YOUR JPEXS, then run the SWF again in your flash player or whatever you're using. Log in.

Pt. 8
-

Go to `settings558b.ini` to spinner texts. Add `&sSpinnerText55=Moderator&`
Go to `upgradevalues558b.ini` and add `&sSpinnerCost55=1000&` to spinner values.

The Moderator Spinner as you can see is now purchasable in the shop in slot #55. If you attempt to purchase the spinner it will say that it was purchased if you had enough creds, as described in earlier parts, but you will notice it is not in your profile. We need to now add the item ID to our database.

I use phpmyadmin. Navigate to phpmyadmin from XAMMP, navigate to shop. Insert an itemID of 155 and a cost of 1000, or however much the cost is. 

Example query:

`INSERT INTO stick_arena (id, itemID, cost) VALUES (replaceThisWithAnActualId, 155, 1000);`

Pt. 9
-

Now if you want you can get a backup indicatorMC83 from a different SWF file and replace the old Moderator to be MC83 again instead of reverting the current one. Now you'll essentially have 2 moderator spinners, one being the original and one being changed to MC55, but that's fine. Modify this spinner to any custom spinner you'd like, or delete it. You can learn how to modify sprites in previous guides. Be sure to update your slot number from 55 to whatever it actually is supposed to correspond to in the store.


-End

# Future Guides:

* Change Head Color in-game with key presses [super-expert]

![Recording17-ezgif com-crop](https://github.com/Ebubekir-Tas/Flash-Modding-Guide/assets/65694925/324db961-bb6a-47a9-952e-875e506e2320)
