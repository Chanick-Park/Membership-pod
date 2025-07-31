#### AccountViewModel Sequence Diagram 
---
#### DMCCard flow
```mermaid
sequenceDiagram
    participant UI as User
    participant View as AccountView
    participant VM as AccountViewModel
    participant Coordinator as DMCCoordinator
    participant Remote as AccountRemoteDataSource
    participant Repo as AccountRepository

    %% DMC Card Load on View Appear
    UI->>View: Opens AccountView
    View->>VM: .onAppear (AccountContenBody)
    VM->>VM: revalidateDMCs()
    VM->>VM: loadDMCCards()
    VM->>Remote: checkDMCAndUpdateIfNecessary()
    Remote-->>VM: DMC Card(s) (async)
    alt Cards found
        VM->>VM: self.cards = cards
        VM->>VM: publishDMCState(.loaded(cards))
        VM->>Coordinator: updateCards(cards)
        VM->>View: Update UI (DMCView)
    else No cards
        alt authStatus == .loggedIn
            VM->>VM: publishDMCState(.failed(DMCError.dmcDataNotFound))
            VM->>View: Show DMCGenericView
        else
            VM->>VM: publishDMCState(.signedOut)
            VM->>View: Show DMCSignedOutView
        end
    end

    %% DMC Card View State Handling
    View->>View: accountDMCView()
    alt dmcState == .idle
        View->>View: Show Color.clear
    else dmcState == .failed
        View->>View: Show DMCGenericView(viewModel)
    else dmcState == .loading
        View->>View: Show progressView()
    else dmcState == .loaded
        View->>View: Show DMCView(cards, viewModel)
    else dmcState == .signedOut
        View->>View: Show DMCSignedOutView(viewModel)
    end

    %% Menu Tap for DMC Card Features
    UI->>View: Tap DMC Card-related menu
    View->>VM: handleNativeScreenTap(cellData)
    VM->>VM: checkIfDMCValidationCallNecessary(cellData)
    alt DMC Validation Needed
        VM->>VM: revalidateDMCs(skipGlobalValidationCheck: true)
    end
    alt cellData.feature == membershipDetails
        VM->>VM: presentMembershipDetailsView(cellData)
        VM->>View: Navigate to MembershipDetailsScreen
    else cellData.feature == rewards
        VM->>VM: presentRewardsView(cellData)
        VM->>View: Navigate to RewardsScreen
    else cellData.feature == addPayment
        VM->>VM: addPaymentView()
        VM->>View: Navigate to AddPaymentScreen
    end

    %% DMC Card Exit Message (Alert)
    VM->>View: showAlert = true (DMC Exit)
    View->>View: Show Alert (title, message, button)
    UI->>View: Dismiss alert
    View->>VM: resetDMCExitMessage()
```
---
#### membershipStatus update flow.
```mermaid
sequenceDiagram
    participant VM as AccountViewModel
    participant Remote as AccountRemoteDataSource
    participant Store as Store<CostcoAppState>

    %% 1. On DMC Card Data Load (e.g. after validation or login)
    VM->>Remote: checkDMCAndUpdateIfNecessary()
    Remote-->>VM: [dmcList]
    VM->>VM: saveMembershipStatus(isUserLoggedIn, dmcList)
    VM->>Store: Update store.state.memberShipStatus

    %% 2. On User Login
    VM->>Store: UserStateAction.login
    VM->>VM: saveSharedDMC(isUserLoggedIn: true, dmcList: [])
    VM->>VM: saveMembershipStatus(isUserLoggedIn: true, dmcList: [])
    VM->>Store: Update store.state.memberShipStatus

    %% 3. On User Logout
    VM->>Store: UserStateAction.logout
    VM->>VM: saveSharedDMC(isUserLoggedIn: false, dmcList: [])
    VM->>VM: saveMembershipStatus(isUserLoggedIn: false, dmcList: [])
    VM->>Store: Update store.state.memberShipStatus

    %% 4. On DMC Card Change (e.g. after deleting membership card, revalidation)
    VM->>Remote: deleteMembershipCard()
    Remote-->>VM: []
    VM->>VM: saveMembershipStatus(isUserLoggedIn, [])
    VM->>Store: Update store.state.memberShipStatus
```
#### Explanation:

 - saveMembershipStatus is called after DMC card data changes (validation, delete), on user login/logout, and anytime DMC cards are updated.
 - Each call updates store.state.memberShipStatus, reflecting the current membership validity.
 - The diagram tracks these trigger points and their flow in the app.
---
#### revalidationDMC call flow
```mermaid
sequenceDiagram
    participant VM as AccountViewModel
    participant Store as Store<CostcoAppState>
    participant Remote as AccountRemoteDataSource

    %% 1. On App Life Cycle Change (Foreground)
    Store->>VM: appLifecycle = .foreground
    VM->>VM: revalidateDMCs()
    
    %% 2. On Region Change
    Store->>VM: currentRegion changed
    VM->>VM: revalidateDMCs()
    
    %% 3. On AccountView appear
    View->>VM: .onAppear
    VM->>VM: revalidateDMCs()
    
    %% 4. On Passkey/Card/Sign&Security Tiles Refresh
    VM->>VM: updatePasskeyRelatedTiles()/updatePasskeyCardMenuItemIfNeeded()/updateSignAndSecurityMenuItemIfNeeded()
    VM->>VM: revalidateDMCs()
    
    %% 5. On User Login/Logout
    Store->>VM: UserStateAction.login/logout
    VM->>VM: revalidateDMCs()
    
    %% 6. On DMC Validation Needed (menu tap triggers)
    VM->>VM: checkIfDMCValidationCallNecessary(cellData)
    alt Validation Needed
        VM->>VM: revalidateDMCs(skipGlobalValidationCheck: true)
    end
```
#### Explanation:
This sequence shows all situations where revalidateDMCs() is called in the app:

 - App lifecycle event
- Region change
- AccountView appearance
- Passkey/tile refresh
- User login/logout
- DMC validation trigger via menu tap
