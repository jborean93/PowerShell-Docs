---
external help file: Microsoft.PowerShell.Commands.Utility.dll-Help.xml
Locale: en-US
Module Name: Microsoft.PowerShell.Utility
ms.date: 01/02/2024
online version: https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/update-list?view=powershell-7.4&WT.mc_id=ps-gethelp
schema: 2.0.0
title: Update-List
---

# Update-List

## SYNOPSIS
Adds items to and removes items from a property value that contains a collection of objects.

## SYNTAX

### AddRemoveSet (Default)

```
Update-List [-Add <Object[]>] [-Remove <Object[]>] [-InputObject <PSObject>] [[-Property] <String>]
 [<CommonParameters>]
```

### ReplaceSet

```
Update-List -Replace <Object[]> [-InputObject <PSObject>] [[-Property] <String>]
 [<CommonParameters>]
```

## DESCRIPTION

The `Update-List` cmdlet adds, removes, or replaces items in a property value of an object and
returns the updated object. This cmdlet is designed for properties that contain collections of
objects.

The **Add** and **Remove** parameters add individual items to and remove them from the collection.
The **Replace** parameter replaces the entire collection.

If you don't specify a property in the command, `Update-List` returns a hashtable that describes the
update instead of updating the object. Later, you can use this change set to update a list object.

This cmdlet works only when the property that's being updated supports the **IList** interface that
`Update-List` uses. Also, any `Set` cmdlets that accept an update must support the **IList**
interface.

This cmdlet was reintroduced in PowerShell 7.

## EXAMPLES

### Example 1: Add items to a property value

In this example we create a class that represents a deck of cards where the cards are stored as a
**List** collection object. The **NewDeck()** method uses `Update-List`to add a complete deck of
card values to the **Cards** collection.

```powershell
class Cards {

    [System.Collections.Generic.List[string]]$Cards
    [string]$Name

    Cards([string]$_name) {
        $this.Name = $_name
        $this.Cards = [System.Collections.Generic.List[string]]::new()
    }

    NewDeck() {
        $_suits = "`u{2663}","`u{2666}","`u{2665}","`u{2660}"
        $_values = 'A',2,3,4,5,6,7,8,9,10,'J','Q','K'
        $_deck = foreach ($s in $_suits){ foreach ($v in $_values){ "$v$s"} }
        $this | Update-List -Property Cards -Add $_deck | Out-Null
    }

    Show() {
        Write-Host
        Write-Host $this.Name ": " $this.Cards[0..12]
        if ($this.Cards.Count -gt 13) {
            Write-Host (' ' * ($this.Name.Length+3)) $this.Cards[13..25]
        }
        if ($this.Cards.Count -gt 26) {
            Write-Host (' ' * ($this.Name.Length+3)) $this.Cards[26..38]
        }
        if ($this.Cards.Count -gt 39) {
            Write-Host (' ' * ($this.Name.Length+3)) $this.Cards[39..51]
        }
    }

    Shuffle() { $this.Cards = Get-Random -InputObject $this.Cards -Count 52 }

    Sort() { $this.Cards.Sort() }
}
```

> [!NOTE]
> The `Update-List` cmdlet outputs the updated object to the pipeline. We pipe the output to
> `Out-Null` to suppress the unwanted display.

### Example 2: Add and remove items of a collection property

Continuing with the code in Example 1, we will create instances of the **Cards** class to represent
a deck of cards and the cards held by two players. We use the `Update-List` cmdlet to add cards to
the players' hands and to remove cards from the deck.

```powershell
$player1 = [Cards]::new('Player 1')
$player2 = [Cards]::new('Player 2')

$deck = [Cards]::new('Deck')
$deck.NewDeck()
$deck.Shuffle()
$deck.Show()

# Deal two hands
$player1 | Update-List -Property Cards -Add $deck.Cards[0,2,4,6,8] | Out-Null
$player2 | Update-List -Property Cards -Add $deck.Cards[1,3,5,7,9] | Out-Null
$deck | Update-List -Property Cards -Remove $player1.Cards | Out-Null
$deck | Update-List -Property Cards -Remove $player2.Cards | Out-Null

$player1.Show()
$player2.Show()
$deck.Show()
```

```Output
Deck :  4♦ 7♥ J♦ 5♣ A♣ 8♦ J♣ Q♥ 6♦ 3♦ 9♦ 6♣ 2♣
        K♥ 4♠ 10♥ 8♠ 10♦ 9♠ 6♠ K♦ 7♣ 3♣ Q♣ A♥ Q♠
        3♥ 5♥ 2♦ 5♠ J♥ J♠ 10♣ 4♥ Q♦ 10♠ 4♣ 2♠ 2♥
        6♥ 7♦ A♠ 5♦ 8♣ 9♥ K♠ 7♠ 3♠ 9♣ A♦ K♣ 8♥

Player 1 :  4♦ J♦ A♣ J♣ 6♦

Player 2 :  7♥ 5♣ 8♦ Q♥ 3♦

Deck :  9♦ 6♣ 2♣ K♥ 4♠ 10♥ 8♠ 10♦ 9♠ 6♠ K♦ 7♣ 3♣
        Q♣ A♥ Q♠ 3♥ 5♥ 2♦ 5♠ J♥ J♠ 10♣ 4♥ Q♦ 10♠
        4♣ 2♠ 2♥ 6♥ 7♦ A♠ 5♦ 8♣ 9♥ K♠ 7♠ 3♠ 9♣
        A♦ K♣ 8♥
```

The output shows the state of the deck before the cards were dealt to the players. You can see that
each player received five cards from the deck. The final output shows the state of the deck after
dealing the cards to the players. `Update-List` was used to select the cards from the deck and add
them to the players' collection. Then the players' cards were removed from the deck using
`Update-List`.

### Example 3: Add and remove items in a single command

`Update-List` allows you to use the **Add** and **Remove** parameters in a single command. In this
example, Player 1 wants to discard the `4♦` and `6♦` and get two new cards.

```powershell
# Player 1 wants two new cards - remove 2 cards & add 2 cards
$player1 | Update-List -Property Cards -Remove $player1.Cards[0,4] -Add $deck.Cards[0..1] | Out-Null
$player1.Show()

# remove dealt cards from deck
$deck | Update-List -Property Cards -Remove $deck.Cards[0..1] | Out-Null
$deck.Show()
```

```Output
Player 1 :  J♦ A♣ J♣ 9♦ 6♣

Deck :  2♣ K♥ 4♠ 10♥ 8♠ 10♦ 9♠ 6♠ K♦ 7♣ 3♣ Q♣ A♥
        Q♠ 3♥ 5♥ 2♦ 5♠ J♥ J♠ 10♣ 4♥ Q♦ 10♠ 4♣ 2♠
        2♥ 6♥ 7♦ A♠ 5♦ 8♣ 9♥ K♠ 7♠ 3♠ 9♣ A♦ K♣
        8♥
```

### Example 4: Apply a change set to a list object

If you don't specify a property, `Update-List` returns a hashtable that describes the update instead
of updating the object. You can cast the hashtable to a **System.PSListModifier** object and use the
`ApplyTo()` method to apply the change set to a list.

```powershell
$list = [System.Collections.ArrayList] (1, 43, 2)
$changeInstructions = Update-List -Remove 43 -Add 42
$changeInstructions
```

```Output
Name                           Value
----                           -----
Add                            {42}
Remove                         {43}
```

```powershell
([pslistmodifier]($changeInstructions)).ApplyTo($list)
$list
```

```Output
1
2
42
```

## PARAMETERS

### -Add

Specifies the property values to be added to the collection. Enter the values in the order that they
should appear in the collection.

```yaml
Type: System.Object[]
Parameter Sets: AddRemoveSet
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -InputObject

Specifies the objects to be updated. You can also pipe the object to be updated to `Update-List`.

```yaml
Type: System.Management.Automation.PSObject
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: True (ByValue)
Accept wildcard characters: False
```

### -Property

Specifies the property that contains the collection that's being updated. If you omit this
parameter, `Update-List` returns an object that represents the change instead of changing the
object.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: 0
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -Remove

Specifies the property values to be removed from the collection.

```yaml
Type: System.Object[]
Parameter Sets: AddRemoveSet
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -Replace

Specifies a new collection. This parameter replaces all items in the original collection with the
items specified by this parameter.

```yaml
Type: System.Object[]
Parameter Sets: ReplaceSet
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### CommonParameters

This cmdlet supports the common parameters: -Debug, -ErrorAction, -ErrorVariable,
-InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose,
-WarningAction, and -WarningVariable. For more information, see
[about_CommonParameters](https://go.microsoft.com/fwlink/?LinkID=113216).

## INPUTS

### System.Management.Automation.PSObject

You can pipe the object to be updated to this cmdlet.

## OUTPUTS

### System.Collections.Hashtable

By default, this cmdlet returns a hashtable that describes the update.

### System.Object

When you specify the **Property** parameter, this cmdlet returns the updated object.

## NOTES

## RELATED LINKS

[Select-Object](Select-Object.md)
