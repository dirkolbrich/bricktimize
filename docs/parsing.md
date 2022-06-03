# Parsing different file formats for brick information

This is an overview over the different file formats used for brick inventories.

## Bricklink Wanted List

You can export your wanted list from Bricklink in XML format:

https://www.bricklink.com/help.asp?helpID=207

Example:

```xml
<INVENTORY>
  <ITEM>
    <ITEMTYPE>P</ITEMTYPE>
    <ITEMID>3001</ITEMID>
    <COLOR>5</COLOR>
    <MAXPRICE>1.00</MAXPRICE>
    <MINQTY>100</MINQTY>
    <CONDITION>N</CONDITION>
    <REMARKS>for MOC AB154A</REMARKS>
    <NOTIFY>N</NOTIFY>
  </ITEM>
</INVENTORY>
```

##  Brickstore

XML File format of the Brickstore inventory software

Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE BrickStoreXML>
<BrickStoreXML>
<Inventory>
  <Item>
    <ItemID>3010</ItemID>
    <ItemTypeID>P</ItemTypeID>
    <ColorID>34</ColorID>
    <ItemName>Brick 1 x 4</ItemName>
    <ColorName>Lime</ColorName>
    <Qty>12</Qty>
    <Price>0</Price>
    <Condition>X</Condition>
  </Item>
</Inventory>
</BrickStoreXML>
```

## Brickset

downloadable parts inventory as .csv file

Example:

```csv
Part,Color,Quantity,Is Spare
3010,5,10,False
```
