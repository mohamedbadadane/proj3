# Q.5.7 (Importation du module Functions.psm1 pour utiliser la fonction log)
Import-Module "C:\Scripts\Functions.psm1"

Function Random-Password
{
    param ([Int]$Length = 8)
    
    $Punc = 46..46
    $Digits = 48..57
    $Letters = 65..90 + 97..122
    $Password = Get-Random -Count $Length -Input ($Punc + $Digits + $Letters) |`
        ForEach -begin { $aa = $null } -process {$aa += [char]$_} -end {$aa}
    Return $Password.ToString()
}

Function ManageAccentsAndCapitalLetters
{
    param ([String]$String)
    
    $StringWithoutAccent = $String -replace '[éèêë]', 'e' -replace '[àâä]', 'a' -replace '[îï]', 'i' -replace '[ôö]', 'o' -replace '[ùûü]', 'u'
    $StringWithoutAccentAndCapitalLetters = $StringWithoutAccent.ToLower()
    $StringWithoutAccentAndCapitalLetters
}

$Path = "C:\Scripts"
$CsvFile = "$Path\Users.csv"
$LogFile = "$Path\Log.log"
# Q.5.3 (Suppression du Header et du Skip -2 pour prise en compte création du 1er user)
# Q.5.5 (Modification pour que seul les champs utilisés soient importés : Select-Object -Property prenom, nom, description, fonction)
$Users = Import-Csv -Path $CsvFile -Delimiter ";" `
    -Header "prenom","nom","fonction","description" `
    -Encoding UTF8 | Select-Object -Skip 1
     
foreach ($User in $Users)
{
    $Prenom = ManageAccentsAndCapitalLetters -String $User.prenom
    $Nom = ManageAccentsAndCapitalLetters -String $User.Nom
    $Name = "$Prenom.$Nom"
    If (-not(Get-LocalUser -Name "$Prenom.$Nom" -ErrorAction SilentlyContinue))
    {
	$Pass = Random-Password
        $Password = (ConvertTo-secureString $Pass -AsPlainText -Force)
        $Description = "$($User.Description) - $($User.Fonction)"
        # Q.5.4 (Ajout de la variable $Description dans $UserInfo pour utiliser le champ Description dans fichier Users.csv)
        # Q.5.11 (On met PasswordNeverExpires sur $true au lieu de $False pour que le mot de passe soit permanent)
        $UserInfo = @{
            Name                 = "$Prenom.$Nom"
            FullName             = "$Prenom.$Nom"
            Password             = $Password
	    Description          = $Description	
            AccountNeverExpires  = $True
            PasswordNeverExpires = $True
        }

        New-LocalUser @UserInfo
        #Q.5.10 (Ajout du "s" à Utilisateur)
        Add-LocalGroupMember -Group "Utilisateurs" -Member $Name 
        # Q.5.6 (Ajout du code : avec le mot de passe $Password" -Foreground Green à la fin du Write-Host)
        Write-Host "L'utilisateur $Name a été crée avec le mot de passe $Pass" -ForegroundColor Green
	# Q.5.8 (Message sucès ajouté dans le Log.log)
	Log -FilePath $LogFile -Content "Création de l'utilisateur : $Name"
    }
    Else {
	Write-Host "Le compte $Name existe déjà" -ForeGroundColor Red
    }
    # Q.5.9 (Message d'erreur en rouge si l'utilisateur existe déjà)
}
