# Gameplay Ability
## Define Trigger Tag in CPP

```
if (HasAnyFlags(RF_ClassDefaultObject))  
{  
    // Add the ability trigger tag as default to the CDO.  
    FAbilityTriggerData TriggerData;  
    TriggerData.TriggerTag = LyraGameplayTags::GameplayEvent_Death;  
    TriggerData.TriggerSource = EGameplayAbilityTriggerSource::GameplayEvent;  
    AbilityTriggers.Add(TriggerData);  
}
```


## Activation Group

### Enum def

```
/**  
 * ELyraAbilityActivationGroup * *  Defines how an ability activates in relation to other abilities. */UENUM(BlueprintType)  
enum class ELyraAbilityActivationGroup : uint8  
{  
    // Ability runs independently of all other abilities.  
    Independent,  
  
    // Ability is canceled and replaced by other exclusive abilities.  
    Exclusive_Replaceable,  
  
    // Ability blocks all other exclusive abilities from activating.  
    Exclusive_Blocking,  
  
    MAX    UMETA(Hidden)  
};
```

### ASC Management:

```
IsActivationGroupBlocked(ELyraAbilityActivationGroup Group) const;  
void AddAbilityToActivationGroup(ELyraAbilityActivationGroup Group, ULyraGameplayAbility* LyraAbility);  
void RemoveAbilityFromActivationGroup(ELyraAbilityActivationGroup Group, ULyraGameplayAbility* LyraAbility);  
void CancelActivationGroupAbilities(ELyraAbilityActivationGroup Group, ULyraGameplayAbility* IgnoreLyraAbility, bool bReplicateCancelAbility);

// Number of abilities running in each activation group.  
int32 ActivationGroupCounts[(uint8)ELyraAbilityActivationGroup::MAX];
```

The ActivationGroupCounts is mainly used to keep track of the number of exclusive abilities to ensure that only one exclusive ability is running.

### Ability Management

1. Ability decides its activation group and changing to different activation group

```
// Returns true if the requested activation group is a valid transition.  
UFUNCTION(BlueprintCallable, BlueprintPure = false, Category = "Lyra|Ability", Meta = (ExpandBoolAsExecs = "ReturnValue"))  
bool CanChangeActivationGroup(ELyraAbilityActivationGroup NewGroup) const;  
  
// Tries to change the activation group.  Returns true if it successfully changed.  
UFUNCTION(BlueprintCallable, BlueprintPure = false, Category = "Lyra|Ability", Meta = (ExpandBoolAsExecs = "ReturnValue"))  
bool ChangeActivationGroup(ELyraAbilityActivationGroup NewGroup);
```

2. Using the activation group the ability knows if it's activatable in `CanActivateAbility`:
```
	bool ULyraGameplayAbility::CanActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayTagContainer* SourceTags, const FGameplayTagContainer* TargetTags, FGameplayTagContainer* OptionalRelevantTags) const  
{  
    if (!ActorInfo || !ActorInfo->AbilitySystemComponent.IsValid())  
    {       return false;  
    }  
    if (!Super::CanActivateAbility(Handle, ActorInfo, SourceTags, TargetTags, OptionalRelevantTags))  
    {       return false;  
    }  
    //@TODO Possibly remove after setting up tag relationships  
    ULyraAbilitySystemComponent* LyraASC = CastChecked<ULyraAbilitySystemComponent>(ActorInfo->AbilitySystemComponent.Get());  
    if (LyraASC->IsActivationGroupBlocked(ActivationGroup))  
    {       if (OptionalRelevantTags)  
       {          OptionalRelevantTags->AddTag(LyraGameplayTags::Ability_ActivateFail_ActivationGroup);  
       }       return false;  
    }  
    return true;  
}
```

