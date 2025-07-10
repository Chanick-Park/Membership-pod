#### AccountViewModel Sequence Diagram for DMCCard
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
