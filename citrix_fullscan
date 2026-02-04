<#
    .SYNOPSIS
    Citrix Ultimate Inventory Tool v6.0 (All-in-One)
    .DESCRIPTION
    Universal script (Modules + Snapins).
    Features: Dark GUI, Search, User Name, Session Count, Status, Maintenance, Hypervisor.
#>

# =============================================================================
# 1. SETUP & GLOBAL VARS
# =============================================================================
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing
Add-Type -AssemblyName System.Data

# Global DataTable required for the Search/Filter logic
$global:dt = New-Object System.Data.DataTable

# =============================================================================
# 2. LOGIC FUNCTIONS
# =============================================================================

function Log-Message {
    param($Msg, $Color="White")
    $timestamp = Get-Date -Format "HH:mm:ss"
    $rtbConsole.SelectionStart = $rtbConsole.TextLength
    $rtbConsole.SelectionColor = [System.Drawing.Color]::FromName($Color)
    $rtbConsole.AppendText("$timestamp - $Msg`r`n")
    $rtbConsole.ScrollToCaret()
    [System.Windows.Forms.Application]::DoEvents()
}

function Get-CitrixData {
    $btnRun.Enabled = $false
    $txtSearch.Enabled = $false
    $txtSearch.Text = ""
    
    Log-Message "--- STARTING COMPLETE SCAN ---" "Cyan"

    # A. Load Components (Universal)
    $loaded = $false
    if (Get-Module -ListAvailable "Citrix.Broker.Admin.V2") {
        Log-Message "Loading Citrix Modules..." "Gray"
        Import-Module Citrix.Broker.Admin.V2 -ErrorAction SilentlyContinue
        Import-Module Citrix.Common.Commands -ErrorAction SilentlyContinue
        $loaded = $true
    } elseif (Get-PSSnapin -Registered "Citrix*") {
        Log-Message "Loading Citrix Snap-ins..." "Yellow"
        Add-PSSnapin Citrix* -ErrorAction SilentlyContinue
        $loaded = $true
    }

    if (-not $loaded) {
        [System.Windows.Forms.MessageBox]::Show("Citrix SDK not found.", "Error", 0, 16)
        $btnRun.Enabled = $true
        return
    }

    # B. Query Data
    Log-Message "Querying Delivery Controller..." "Green"
    
    try {
        $cursor = [System.Windows.Forms.Cursor]::Current
        [System.Windows.Forms.Cursor]::Current = [System.Windows.Forms.Cursors]::WaitCursor

        # PROPERTIES: Retrieving EVERYTHING requested
        $machines = Get-BrokerMachine -MaxRecordCount 20000 -Property MachineName, CatalogName, DesktopGroupName, AllocationType, Tags, RegistrationState, InMaintenanceMode, SessionUserName, SessionCount, HostingServerName -ErrorAction Stop
        
        Log-Message "Processing $($machines.Count) records..." "Gray"

        # C. Build DataTable
        $global:dt = New-Object System.Data.DataTable
        $global:dt.Columns.Add("Machine Name")
        $global:dt.Columns.Add("User")
        $global:dt.Columns.Add("Sess", [int]) # Session Count (Integer for sorting)
        $global:dt.Columns.Add("Status")       
        $global:dt.Columns.Add("Maint")        
        $global:dt.Columns.Add("Hypervisor")
        $global:dt.Columns.Add("Type")
        $global:dt.Columns.Add("Catalog")
        $global:dt.Columns.Add("Delivery Group")
        
        foreach ($m in $machines) {
            # Logic: User
            $userDisplay = if (-not [string]::IsNullOrEmpty($m.SessionUserName)) { $m.SessionUserName } else { "-" }

            # Logic: Maintenance
            $maintDisplay = if ($m.InMaintenanceMode) { "YES" } else { "No" }

            # Logic: Type
            $typeDisplay = if ($m.AllocationType -eq "Static") { "Personal" } else { "Shared" }

            # Logic: Hypervisor
            $hypDisplay = if (-not [string]::IsNullOrEmpty($m.HostingServerName)) { $m.HostingServerName } else { "-" }

            # Add Row
            $global:dt.Rows.Add(
                $m.MachineName,
                $userDisplay,
                $m.SessionCount,   # <--- Added back as requested
                $m.RegistrationState,
                $maintDisplay,
                $hypDisplay,
                $typeDisplay, 
                $m.CatalogName, 
                $m.DesktopGroupName
            ) | Out-Null
        }

        # Bind to Grid
        $grid.DataSource = $global:dt
        
        # Adjust Column Widths for best fit
        $grid.Columns["Machine Name"].Width = 130
        $grid.Columns["User"].Width = 120
        $grid.Columns["Sess"].Width = 40    # Compact
        $grid.Columns["Status"].Width = 80
        $grid.Columns["Maint"].Width = 50   # Compact
        $grid.Columns["Hypervisor"].Width = 90
        $grid.Columns["Type"].Width = 70

        Log-Message "SUCCESS: Inventory Complete." "Cyan"
        
        $txtSearch.Enabled = $true
        $btnExport.Enabled = $true

    } catch {
        Log-Message "CRITICAL ERROR: $($_.Exception.Message)" "Red"
    } finally {
        [System.Windows.Forms.Cursor]::Current = $cursor
        $btnRun.Enabled = $true
    }
}

function Export-CsvData {
    if ($grid.Rows.Count -eq 0) { return }

    $sfd = New-Object System.Windows.Forms.SaveFileDialog
    $sfd.Filter = "CSV Report (*.csv)|*.csv"
    $sfd.FileName = "Citrix_FullInventory_$(Get-Date -Format 'yyyyMMdd_HHmm').csv"
    
    if ($sfd.ShowDialog() -eq "OK") {
        try {
            $global:dt | Export-Csv -Path $sfd.FileName -NoTypeInformation -Encoding UTF8
            Log-Message "File saved: $($sfd.FileName)" "Green"
            [System.Windows.Forms.MessageBox]::Show("Export Successful!", "Done", 0, 64)
        } catch {
            Log-Message "Export Failed: $($_.Exception.Message)" "Red"
        }
    }
}

function Filter-Grid {
    if ($global:dt.Rows.Count -gt 0) {
        $txt = $txtSearch.Text
        # Search across Machine, User, Catalog OR Hypervisor
        $global:dt.DefaultView.RowFilter = "([Machine Name] LIKE '%$txt%') OR (User LIKE '%$txt%') OR (Catalog LIKE '%$txt%') OR (Hypervisor LIKE '%$txt%')"
    }
}

# =============================================================================
# 3. GUI SETUP (Dark Enterprise Theme)
# =============================================================================
$form = New-Object System.Windows.Forms.Form
$form.Text = "Citrix Ultimate Tool v6.0 (All-in-One)"
$form.Size = New-Object System.Drawing.Size(1300, 800)
$form.StartPosition = "CenterScreen"
$form.BackColor = [System.Drawing.Color]::FromArgb(45, 45, 48) 
$form.ForeColor = "WhiteSmoke"
$form.FormBorderStyle = "Sizable"

# Fonts
$fontTitle = New-Object System.Drawing.Font("Segoe UI", 11, [System.Drawing.FontStyle]::Bold)
$fontLabel = New-Object System.Drawing.Font("Segoe UI", 9, [System.Drawing.FontStyle]::Regular)
$fontInput = New-Object System.Drawing.Font("Segoe UI", 10, [System.Drawing.FontStyle]::Regular)
$fontLog   = New-Object System.Drawing.Font("Consolas", 9, [System.Drawing.FontStyle]::Regular)

# --- LEFT PANEL: INFO ---
$grpInfo = New-Object System.Windows.Forms.GroupBox
$grpInfo.Text = " 1. System Info "
$grpInfo.Location = New-Object System.Drawing.Point(15, 15)
$grpInfo.Size = New-Object System.Drawing.Size(300, 80)
$grpInfo.ForeColor = "Cyan"
$grpInfo.Font = $fontTitle
$form.Controls.Add($grpInfo)

    $lblHost = New-Object System.Windows.Forms.Label
    $lblHost.Text = "Host: $env:COMPUTERNAME | User: $env:USERNAME"
    $lblHost.Location = New-Object System.Drawing.Point(15, 30)
    $lblHost.AutoSize = $true
    $lblHost.ForeColor = "White"
    $lblHost.Font = $fontLabel
    $grpInfo.Controls.Add($lblHost)

# --- LEFT PANEL: ACTIONS ---
$grpAction = New-Object System.Windows.Forms.GroupBox
$grpAction.Text = " 2. Actions "
$grpAction.Location = New-Object System.Drawing.Point(15, 110)
$grpAction.Size = New-Object System.Drawing.Size(300, 150)
$grpAction.ForeColor = "LightGreen"
$grpAction.Font = $fontTitle
$form.Controls.Add($grpAction)

    $btnRun = New-Object System.Windows.Forms.Button
    $btnRun.Text = "► RUN FULL SCAN"
    $btnRun.Location = New-Object System.Drawing.Point(20, 30)
    $btnRun.Size = New-Object System.Drawing.Size(260, 45)
    $btnRun.BackColor = "SeaGreen"
    $btnRun.ForeColor = "White"
    $btnRun.FlatStyle = "Flat"
    $btnRun.FlatAppearance.BorderSize = 0
    $btnRun.Cursor = [System.Windows.Forms.Cursors]::Hand
    $btnRun.Add_Click({ Get-CitrixData })
    $grpAction.Controls.Add($btnRun)

    $btnExport = New-Object System.Windows.Forms.Button
    $btnExport.Text = "💾 EXPORT CSV"
    $btnExport.Location = New-Object System.Drawing.Point(20, 85)
    $btnExport.Size = New-Object System.Drawing.Size(260, 45)
    $btnExport.BackColor = "DimGray"
    $btnExport.ForeColor = "White"
    $btnExport.FlatStyle = "Flat"
    $btnExport.FlatAppearance.BorderSize = 0
    $btnExport.Cursor = [System.Windows.Forms.Cursors]::Hand
    $btnExport.Enabled = $false
    $btnExport.Add_Click({ Export-CsvData })
    $grpAction.Controls.Add($btnExport)

# --- LEFT PANEL: LOG ---
$grpLog = New-Object System.Windows.Forms.GroupBox
$grpLog.Text = " 3. Live Log "
$grpLog.Location = New-Object System.Drawing.Point(15, 275)
$grpLog.Size = New-Object System.Drawing.Size(300, 470)
$grpLog.ForeColor = "Yellow"
$grpLog.Font = $fontTitle
$grpLog.Anchor = "Top, Bottom, Left"
$form.Controls.Add($grpLog)

    $rtbConsole = New-Object System.Windows.Forms.RichTextBox
    $rtbConsole.Dock = "Fill"
    $rtbConsole.BackColor = [System.Drawing.Color]::FromArgb(30, 30, 30)
    $rtbConsole.ForeColor = "LightGray"
    $rtbConsole.Font = $fontLog
    $rtbConsole.ReadOnly = $true
    $rtbConsole.BorderStyle = "None"
    $grpLog.Controls.Add($rtbConsole)


# --- RIGHT PANEL: DATA GRID & SEARCH ---
$grpData = New-Object System.Windows.Forms.GroupBox
$grpData.Text = " Enterprise Inventory "
$grpData.Location = New-Object System.Drawing.Point(330, 15)
$grpData.Size = New-Object System.Drawing.Size(940, 730)
$grpData.ForeColor = "White"
$grpData.Font = $fontTitle
$grpData.Anchor = "Top, Bottom, Left, Right"
$form.Controls.Add($grpData)

    # Search Bar Panel
    $pnlSearch = New-Object System.Windows.Forms.Panel
    $pnlSearch.Dock = "Top"
    $pnlSearch.Height = 50
    $pnlSearch.BackColor = [System.Drawing.Color]::FromArgb(45, 45, 48)
    $grpData.Controls.Add($pnlSearch)

    $lblSearch = New-Object System.Windows.Forms.Label
    $lblSearch.Text = "🔍 Search (User, Machine, Host):"
    $lblSearch.Location = New-Object System.Drawing.Point(10, 15)
    $lblSearch.AutoSize = $true
    $lblSearch.Font = $fontLabel
    $pnlSearch.Controls.Add($lblSearch)

    $txtSearch = New-Object System.Windows.Forms.TextBox
    $txtSearch.Location = New-Object System.Drawing.Point(210, 12)
    $txtSearch.Size = New-Object System.Drawing.Size(400, 26)
    $txtSearch.Font = $fontInput
    $txtSearch.BackColor = [System.Drawing.Color]::FromArgb(30, 30, 30)
    $txtSearch.ForeColor = "White"
    $txtSearch.BorderStyle = "FixedSingle"
    $txtSearch.Enabled = $false
    $txtSearch.Add_TextChanged({ Filter-Grid })
    $pnlSearch.Controls.Add($txtSearch)

    # Data Grid
    $grid = New-Object System.Windows.Forms.DataGridView
    $grid.Dock = "Fill"
    $grid.BackgroundColor = [System.Drawing.Color]::FromArgb(30, 30, 30)
    $grid.ForeColor = "Black" 
    $grid.Font = $fontLabel
    $grid.BorderStyle = "None"
    $grid.RowHeadersVisible = $false
    $grid.AllowUserToAddRows = $false
    $grid.ReadOnly = $true
    $grid.SelectionMode = "FullRowSelect"
    $grid.AutoSizeColumnsMode = "Fill"
    
    # Grid Styling
    $grid.EnableHeadersVisualStyles = $false
    $grid.ColumnHeadersDefaultCellStyle.BackColor = [System.Drawing.Color]::FromArgb(60, 60, 60)
    $grid.ColumnHeadersDefaultCellStyle.ForeColor = "White"
    $grid.ColumnHeadersBorderStyle = "None"
    $grid.ColumnHeadersHeight = 35
    
    $grid.DefaultCellStyle.BackColor = [System.Drawing.Color]::FromArgb(45, 45, 48)
    $grid.DefaultCellStyle.ForeColor = "WhiteSmoke"
    $grid.DefaultCellStyle.SelectionBackColor = [System.Drawing.Color]::SeaGreen
    $grid.DefaultCellStyle.SelectionForeColor = "White"
    
    $grpData.Controls.Add($grid)
    $grid.BringToFront()

# --- STARTUP ---
Log-Message "Master Tool v6.0 (All-in-One) Ready." "Gray"

[void]$form.ShowDialog()
