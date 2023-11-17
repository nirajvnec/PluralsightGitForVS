// This is the actual event handler that matches the expected signature for event handlers.
private async void CtlDragDropItem_DisabledChanged_Handler(object sender, EventArgs e)
{
    if (sender is CtlDragDropItem dragDropItem)
    {
        await CtlDragDropItem_DisabledChanged(dragDropItem);
    }
}


// To unsubscribe
new_item.DisabledChanged -= CtlDragDropItemDisabledChanged;

// To subscribe
new_item.DisabledChanged += async (sender, e) => await CtlDragDropItemDisabledChanged(sender, e);



public void RedrawItems(IProgress<int> progress)
{
    var updates = new List<Action>();
    bool itemFoundInHeader = false;
    string currentHeading = m_current_heading_button == null ? string.Empty : m_current_heading_button.Text;
    int labelTop = SIZE_ITEM_SPACING;

    // Suspend layout logic for both the label area and the drag-drop items
    label_area_panel.SuspendLayout();
    m_drag_drop_items.SuspendLayout();

    int totalItems = m_drag_drop_items.Count;
    int processedItems = 0;

    foreach (CtlDragDropItem dragDropItem in m_drag_drop_items)
    {
        string header = dragDropItem.Header.ToUpper();
        bool isCurrentHeading = header == currentHeading.ToUpper();
        bool isDragDropItemVisible = !dragDropItem.Disabled && !dragDropItem.IsDraggedAway;
        bool isItemInCurrentHeading = (isCurrentHeading || currentHeading == CsBreakDownHeading.HEADING_ALL + CsBreakDownHeading.HEADING_SUFFIX) && isDragDropItemVisible;

        // Prepare the updates
        if (isItemInCurrentHeading)
        {
            int currentLabelTop = labelTop;
            updates.Add(() => 
            {
                dragDropItem.Top = currentLabelTop;
                dragDropItem.Visible = true;
            });
            labelTop += SIZE_ITEM_SPACING + dragDropItem.Height;
            itemFoundInHeader = true;
        }
        else
        {
            updates.Add(() => dragDropItem.Visible = false);
        }

        processedItems++;
        int progressPercentage = (processedItems * 100) / totalItems;
        progress?.Report(progressPercentage);
    }

    // Perform all the UI updates using BeginInvoke
    foreach (var update in updates)
    {
        this.BeginInvoke(new MethodInvoker(update));
    }

    // Continue on the UI thread to resume layout and update the panel height
    this.BeginInvoke(new MethodInvoker(() =>
    {
        label_area_panel.Height = labelTop;
        empty_text_label.Visible = !itemFoundInHeader;
        if (!itemFoundInHeader)
        {
            empty_text_label.Top = labelTop;
        }

        // Resume layout logic
        m_drag_drop_items.ResumeLayout(false);
        label_area_panel.ResumeLayout(false);
    }));

    // Final report for progress
    progress?.Report(100);
}







private async void CtlDragDropItemDisabledChanged(object sender, DragDropItemEventArgs e)
{
    // You can await an asynchronous method directly
    await Method1();

    // Or if you need to run something on a background thread and await its completion:
    await Task.Run(async () => 
    {
        // Some background work here. If Method1 is already async, you don't need Task.Run.
        await Method1();
    });

    // More code can follow here and it will run after the awaited Task has completed.
}

private async Task Method1()
{
    // Simulate an async operation
    await Task.Delay(1000);
}





private void OtherFilterGridRiskTypesChanged(object sender, EventArgs e)
{
    // Call the async method without awaiting it
    Task.Run(async () => await SetRiskTypeAttributesAsync(attributes));
}

// Assuming SetRiskTypeAttributesAsync is your async Task method
private async Task SetRiskTypeAttributesAsync(StringCollection attributes)
{
    // Your async code here
}



public void RedrawItems(IProgress<int> progress)
{
    // Gather all the updates first without directly modifying the UI controls
    var updates = new List<Action>();
    bool itemFoundInHeader = false;
    string currentHeading = m_current_heading_button == null ? string.Empty : m_current_heading_button.Text;
    int labelTop = SIZE_ITEM_SPACING;

    int totalItems = m_drag_drop_items.Count;
    int processedItems = 0;

    foreach (CtlDragDropItem dragDropItem in m_drag_drop_items)
    {
        string header = dragDropItem.Header.ToUpper();
        bool isCurrentHeading = header == currentHeading.ToUpper();
        bool isDragDropItemVisible = !dragDropItem.Disabled && !dragDropItem.IsDraggedAway;
        bool isItemInCurrentHeading = (isCurrentHeading || currentHeading == CsBreakDownHeading.HEADING_ALL + CsBreakDownHeading.HEADING_SUFFIX) && isDragDropItemVisible;

        // Prepare the updates
        if (isItemInCurrentHeading)
        {
            int currentLabelTop = labelTop;
            updates.Add(() => 
            {
                dragDropItem.Top = currentLabelTop;
                dragDropItem.Visible = true;
            });
            labelTop += SIZE_ITEM_SPACING + dragDropItem.Height;
            itemFoundInHeader = true;
        }
        else
        {
            updates.Add(() => dragDropItem.Visible = false);
        }

        processedItems++;
        int progressPercentage = (processedItems * 100) / totalItems;
        progress?.Report(progressPercentage);
    }

    // Apply all updates in a single BeginInvoke call
    this.BeginInvoke(new MethodInvoker(() =>
    {
        foreach (var update in updates)
        {
            update.Invoke();
        }

        label_area_panel.Height = labelTop;
        empty_text_label.Visible = !itemFoundInHeader;
        if (!itemFoundInHeader)
        {
            empty_text_label.Top = labelTop;
        }

        m_drag_drop_items.ResumeLayout(false);
        this.ResumeLayout(false);
    }));

    // Report completion, consider marshaling back to the UI thread if called from a background thread
    if (this.InvokeRequired)
    {
        this.BeginInvoke(new MethodInvoker(() =>
        {
            progress?.Report(100);
        }));
    }
    else
    {
        progress?.Report(100);
    }
}








// Assuming you have already defined and initialized your TableLayoutPanel
// and it's named tableLayoutPanel1 in your form designer code

// Adding a new ProgressBar control
ProgressBar progressBar = new ProgressBar();
progressBar.Name = "progressBar";
progressBar.Dock = DockStyle.Fill;

// Add a new RowStyle for the ProgressBar at the second row (index 1)
// If your TableLayoutPanel does not have a second row, you need to add one
tableLayoutPanel1.RowStyles.Add(new RowStyle(SizeType.AutoSize));
tableLayoutPanel1.RowCount = 2; // Set the row count to 2
tableLayoutPanel1.Controls.Add(progressBar, 0, 1); // Add progressBar to the new row

// Set the column span if you want the ProgressBar to span across multiple columns
tableLayoutPanel1.SetColumnSpan(progressBar, tableLayoutPanel1.ColumnCount);

// Now you have your ProgressBar added to the second row of your TableLayoutPanel







private async void m_searchControlSearchEvent(object sender, SearchRiskAttributeEventArgs e)
{
    pnlSearch.Visible = true;
    breakdown_attributes.Visible = false;
    breakdown_headings_combo_box.Enabled = false;
    breakdown_headings_combo_box.SelectedIndex = 1;
    DisableBreakdownsByText(m_global_cache.HideBreakdownAndDrilldowns);

    var selectedCalculationMethod = this.GetSelectedMethod();

    if (ctlCalculationMethod.SelectedMethod is CsCalculationContribution && 
        ((CsCalculationContribution)ctlCalculationMethod.SelectedMethod).IsErcType)
    {
        if (rbtnConstant.Checked)
        {
            RemoveColumnRowBreakdown("cobdate", CobDateBreakdown);
            DisableBreakdownByText(CobDateBreakdown, true);
        }
        else
        {
            DisableBreakdownChooseDetailsOption(CobDateBreakdown, false);
            DisableBreakdownByText(RISK_TYPE, false);
        }
    }
    else
    {
        DisableBreakdownByText(CobDateBreakdown, false);
        DisableBreakdownChooseDetailsOption(CobDateBreakdown, true);

        if (!(selectedCalculationMethod is CsCalculationMethodBMCPlStrip) && 
            !(selectedCalculationMethod is CsCalculationMethodRNIVMarginalToVar))
        {
            DisableBreakdownByText(RISK_TYPE, false);
        }
    }

    var progress = new Progress<int>(percent =>
    {
        // Update the UI based on progress
        // For example, updating a progress bar
        // progressBar.Value = percent;
    });

    // Modify the SearchAttributes method to support Progress<T> and call it here
    await SearchAttributes(e, progress);

    breakdown_attributes.Visible = true;
    pnlSearch.Visible = false;
}



private async Task SearchAttributes(SearchRiskAttributeEventArgs e, IProgress<int> progress)
{
    string searchString = e.RiskAttributeName.ToUpper();
    bool resultFound = await Task.Run(() => RedrawSearchItems(searchString, progress));

    // Update the UI based on the result of the search
    if (!resultFound)
    {
        this.Invoke(new Action(() =>
        {
            empty_text_label.Visible = true;
            empty_text_label.Text = "No matching attributes found...";
        }));
    }
    else
    {
        this.Invoke(new Action(() =>
        {
            empty_text_label.Visible = false;
        }));
    }
}


public bool RedrawSearchItems(string searchstring, IProgress<int> progress)
{
    int label_top = SIZE_ITEM_SPACING;
    bool resultfound = false;
    string[] searchStringArray = searchstring.Split(new Char[] { ' ', '+', '-', '_', '(', ')', '#', '@', '{', '}', '[', ']', '*', '|'}, StringSplitOptions.RemoveEmptyEntries);

    this.Invoke(new MethodInvoker(() => {
        this.SuspendLayout();
        label_area_panel.Top = TopOfLabelArea;
        lbl_separator.Visible = false;
        m_drag_drop_items.SuspendLayout();
    }));

    int totalItems = m_drag_drop_items.Count;
    int processedItems = 0;

    foreach (CtlDragDropItem drag_drop_item in m_drag_drop_items)
    {
        bool isSubString = false;
        this.Invoke(new MethodInvoker(() => {
            if (IsHidden(((MarsEnquiryTool.CtlDragDropBreakdown)drag_drop_item))) return;
            if (HideErcNtcsMnpiAttributes(((MarsEnquiryTool.CtlDragDropBreakdown)drag_drop_item))) return;

            if (!drag_drop_item.IsDraggedAway)
            {
                foreach (string searchSubstring in searchStringArray)
                {
                    isSubString = drag_drop_item.Text.ToUpper().StartsWith(searchSubstring.ToUpper());
                    if (isSubString) break;
                }

                if (!drag_drop_item.Disabled && isSubString)
                {
                    drag_drop_item.Top = label_top;
                    label_top += SIZE_ITEM_SPACING + drag_drop_item.Height;
                    drag_drop_item.Visible = true;
                    resultfound = true;
                }
                else
                {
                    drag_drop_item.Visible = false;
                }
            }
        }));

        processedItems++;
        progress?.Report((processedItems * 100) / totalItems);
    }

    this.Invoke(new MethodInvoker(() => {
        m_drag_drop_items.ResumeLayout(false);
        label_area_panel.Height = label_top;
        RedrawScrollArea();
        this.ResumeLayout(false);
    }));

    return resultfound;
}
