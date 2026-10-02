
<img width="904" height="543" alt="image" src="https://github.com/user-attachments/assets/8fd21521-ccca-4600-8f47-6abdc570475c" />


using bai3;
using System;
using System.Collections.Generic;
using System.ComponentModel;  
using System.Drawing;
using System.IO;
using System.Linq;
using System.Text;
using System.Windows.Forms;

namespace bai3
{
    public partial class Form1 : Form
    {
        // =========================================================
        // DỮ LIỆU
        // =========================================================

        private BindingList<Product> products;

        private Product selectedProduct = null;

        private string selectedImagePath = "";

        // =========================================================
        // KHỞI TẠO FORM
        // =========================================================

        public Form1()
        {
            InitializeComponent();

            // Khởi tạo danh sách
            products = new BindingList<Product>();

            // Load ComboBox
            LoadCategories();

            // Binding dữ liệu
            SetupDataBinding();

            // Gắn các sự kiện Click/TextChanged/SelectionChanged
            RegisterEvents();

            // Cập nhật StatusStrip
            UpdateStatus();

            // Không chọn dòng nào lúc đầu
            dgvProducts.ClearSelection();
        }

        // =========================================================
        // COMBOBOX DANH MỤC
        // =========================================================

        private void LoadCategories()
        {
            List<Category> categories = new List<Category>()
            {
                new Category
                {
                    Id = 1,
                    Name = "Điện thoại"
                },

                new Category
                {
                    Id = 2,
                    Name = "Laptop"
                },

                new Category
                {
                    Id = 3,
                    Name = "Phụ kiện"
                }
            };

            // Đúng yêu cầu DisplayMember
            cboCategory.DisplayMember = "Name";

            // Đúng yêu cầu ValueMember
            cboCategory.ValueMember = "Id";

            cboCategory.DataSource = categories;

            cboCategory.SelectedIndex = -1;
        }

        // =========================================================
        // BINDING
        // BindingList<Product> -> BindingSource -> DataGridView
        // =========================================================

        private void SetupDataBinding()
        {
            productBindingSource.DataSource = products;

            dgvProducts.DataSource = productBindingSource;

            dgvProducts.AutoGenerateColumns = false;

            dgvProducts.SelectionMode =
                DataGridViewSelectionMode.FullRowSelect;

            dgvProducts.MultiSelect = false;

            dgvProducts.ReadOnly = true;

            dgvProducts.AllowUserToAddRows = false;

            // TC03
            // Hiển thị 25,000,000
            colUnitPrice.DefaultCellStyle.Format = "N0";

            colUnitPrice.DefaultCellStyle.Alignment =
                DataGridViewContentAlignment.MiddleRight;
        }

        // =========================================================
        // GẮN TOÀN BỘ SỰ KIỆN
        // =========================================================

        private void RegisterEvents()
        {
            // CHỌN ẢNH
            btnChooseImage.Click += btnChooseImage_Click;

            // THÊM
            btnAdd.Click += btnAdd_Click;

            // CẬP NHẬT
            btnUpdate.Click += btnUpdate_Click;

            // XÓA
            btnDelete.Click += btnDelete_Click;

            // CLICK/CHỌN DÒNG DATAGRIDVIEW
            dgvProducts.SelectionChanged +=
                dgvProducts_SelectionChanged;

            // LIVE SEARCH
            txtSearch.TextChanged +=
                txtSearch_TextChanged;

            // EXPORT CSV
            mnuExportCSV.Click +=
                mnuExportCSV_Click;

            // EXIT
            mnuExit.Click +=
                mnuExit_Click;
        }

        // =========================================================
        // VALIDATION
        // TC02
        // =========================================================

        private bool ValidateInput()
        {
            bool valid = true;

            // Xóa lỗi cũ
            errorProvider.Clear();

            // -----------------------------------------------------
            // TÊN SẢN PHẨM
            // -----------------------------------------------------

            if (string.IsNullOrWhiteSpace(
                txtProductName.Text))
            {
                errorProvider.SetError(
                    txtProductName,
                    "Tên sản phẩm không được để trống!");

                valid = false;
            }

            // -----------------------------------------------------
            // ĐƠN GIÁ
            // -----------------------------------------------------

            decimal price;

            if (!decimal.TryParse(
                    txtUnitPrice.Text.Trim(),
                    out price)
                || price <= 0)
            {
                errorProvider.SetError(
                    txtUnitPrice,
                    "Đơn giá phải lớn hơn 0!");

                valid = false;
            }

            // -----------------------------------------------------
            // SỐ LƯỢNG
            // -----------------------------------------------------

            int quantity;

            if (!int.TryParse(
                    txtQuantity.Text.Trim(),
                    out quantity)
                || quantity < 0)
            {
                errorProvider.SetError(
                    txtQuantity,
                    "Số lượng phải >= 0!");

                valid = false;
            }

            return valid;
        }

        // =========================================================
        // CHỌN ẢNH
        // TC04
        // =========================================================

        private void btnChooseImage_Click(
            object sender,
            EventArgs e)
        {
            using (OpenFileDialog openFileDialog =
                   new OpenFileDialog())
            {
                openFileDialog.Title =
                    "Chọn ảnh sản phẩm";

                // Chỉ hiển thị file ảnh
                openFileDialog.Filter =
                    "File ảnh (*.png;*.jpg;*.jpeg;*.bmp)|" +
                    "*.png;*.jpg;*.jpeg;*.bmp";

                openFileDialog.FilterIndex = 1;

                openFileDialog.Multiselect = false;

                if (openFileDialog.ShowDialog()
                    == DialogResult.OK)
                {
                    selectedImagePath =
                        openFileDialog.FileName;

                    try
                    {
                        // Xóa ảnh cũ
                        if (picAvatar.Image != null)
                        {
                            picAvatar.Image.Dispose();

                            picAvatar.Image = null;
                        }

                        // Load ảnh
                        // Cách này không khóa file ảnh
                        using (Image tempImage =
                               Image.FromFile(
                                   selectedImagePath))
                        {
                            picAvatar.Image =
                                new Bitmap(tempImage);
                        }

                        // TC04 yêu cầu Zoom
                        picAvatar.SizeMode =
                            PictureBoxSizeMode.Zoom;
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show(
                            "Không thể mở ảnh!\n" +
                            ex.Message,
                            "Lỗi",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Error);
                    }
                }
            }
        }

        // =========================================================
        // THÊM MỚI
        // =========================================================

        private void btnAdd_Click(
            object sender,
            EventArgs e)
        {
            // TC02:
            // Không hợp lệ thì dừng ngay
            if (!ValidateInput())
            {
                return;
            }

            string productId =
                txtProductId.Text.Trim();

            // -----------------------------------------------------
            // KIỂM TRA MÃ TRÙNG
            // -----------------------------------------------------

            bool duplicate =
                products.Any(p =>
                    p.ProductId.Equals(
                        productId,
                        StringComparison.OrdinalIgnoreCase));

            if (duplicate &&
                !string.IsNullOrWhiteSpace(productId))
            {
                MessageBox.Show(
                    "Mã sản phẩm đã tồn tại!",
                    "Thông báo",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Warning);

                txtProductId.Focus();

                return;
            }

            // -----------------------------------------------------
            // LẤY CATEGORY ID
            // -----------------------------------------------------

            int categoryId = 0;

            if (cboCategory.SelectedValue != null)
            {
                int.TryParse(
                    cboCategory.SelectedValue.ToString(),
                    out categoryId);
            }

            // -----------------------------------------------------
            // TẠO PRODUCT
            // -----------------------------------------------------

            Product product = new Product();

            product.ProductId =
                txtProductId.Text.Trim();

            product.ProductName =
                txtProductName.Text.Trim();

            product.CategoryId =
                categoryId;

            product.CategoryName =
                cboCategory.Text;

            product.UnitPrice =
                decimal.Parse(
                    txtUnitPrice.Text.Trim());

            product.Quantity =
                int.Parse(
                    txtQuantity.Text.Trim());

            product.ImagePath =
                selectedImagePath;

            // -----------------------------------------------------
            // THÊM VÀO BindingList
            // -----------------------------------------------------

            products.Add(product);

            productBindingSource.ResetBindings(false);

            // Status
            UpdateStatus();

            MessageBox.Show(
                "Thêm sản phẩm thành công!",
                "Thông báo",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);

            ClearInput();
        }

        // =========================================================
        // CLICK DÒNG DATAGRIDVIEW
        // -> NẠP NGƯỢC LÊN FORM
        // =========================================================

        private void dgvProducts_SelectionChanged(
            object sender,
            EventArgs e)
        {
            if (dgvProducts.CurrentRow == null)
            {
                return;
            }

            Product product =
                dgvProducts.CurrentRow.DataBoundItem
                as Product;

            if (product == null)
            {
                return;
            }

            selectedProduct = product;

            // Mã SP
            txtProductId.Text =
                product.ProductId;

            // Tên
            txtProductName.Text =
                product.ProductName;

            // Danh mục
            cboCategory.SelectedValue =
                product.CategoryId;

            // Đơn giá
            txtUnitPrice.Text =
                product.UnitPrice.ToString("0");

            // Số lượng
            txtQuantity.Text =
                product.Quantity.ToString();

            // Đường dẫn ảnh
            selectedImagePath =
                product.ImagePath;

            // Load lại ảnh
            LoadProductImage(
                product.ImagePath);
        }

        // =========================================================
        // LOAD ẢNH KHI CLICK DATAGRIDVIEW
        // =========================================================

        private void LoadProductImage(
            string imagePath)
        {
            // Xóa ảnh cũ
            if (picAvatar.Image != null)
            {
                picAvatar.Image.Dispose();

                picAvatar.Image = null;
            }

            if (string.IsNullOrWhiteSpace(
                imagePath))
            {
                return;
            }

            if (!File.Exists(imagePath))
            {
                return;
            }

            try
            {
                using (Image tempImage =
                       Image.FromFile(imagePath))
                {
                    picAvatar.Image =
                        new Bitmap(tempImage);
                }

                picAvatar.SizeMode =
                    PictureBoxSizeMode.Zoom;
            }
            catch
            {
                picAvatar.Image = null;
            }
        }

        // =========================================================
        // CẬP NHẬT
        // =========================================================

        private void btnUpdate_Click(
            object sender,
            EventArgs e)
        {
            // Phải chọn sản phẩm
            if (selectedProduct == null)
            {
                MessageBox.Show(
                    "Vui lòng chọn sản phẩm cần cập nhật!",
                    "Thông báo",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Warning);

                return;
            }

            // Validation
            if (!ValidateInput())
            {
                return;
            }

            string newProductId =
                txtProductId.Text.Trim();

            // -----------------------------------------------------
            // KIỂM TRA MÃ TRÙNG VỚI SẢN PHẨM KHÁC
            // -----------------------------------------------------

            bool duplicate =
                products.Any(p =>
                    p != selectedProduct &&
                    p.ProductId.Equals(
                        newProductId,
                        StringComparison.OrdinalIgnoreCase));

            if (duplicate)
            {
                MessageBox.Show(
                    "Mã sản phẩm đã tồn tại!",
                    "Thông báo",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Warning);

                return;
            }

            int categoryId = 0;

            if (cboCategory.SelectedValue != null)
            {
                int.TryParse(
                    cboCategory.SelectedValue.ToString(),
                    out categoryId);
            }

            // -----------------------------------------------------
            // UPDATE
            // -----------------------------------------------------

            selectedProduct.ProductId =
                txtProductId.Text.Trim();

            selectedProduct.ProductName =
                txtProductName.Text.Trim();

            selectedProduct.CategoryId =
                categoryId;

            selectedProduct.CategoryName =
                cboCategory.Text;

            selectedProduct.UnitPrice =
                decimal.Parse(
                    txtUnitPrice.Text.Trim());

            selectedProduct.Quantity =
                int.Parse(
                    txtQuantity.Text.Trim());

            selectedProduct.ImagePath =
                selectedImagePath;

            // Refresh Grid
            productBindingSource.ResetBindings(false);

            MessageBox.Show(
                "Cập nhật sản phẩm thành công!",
                "Thông báo",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);
        }

        // =========================================================
        // XÓA
        // TC05
        // =========================================================

        private void btnDelete_Click(
            object sender,
            EventArgs e)
        {
            if (selectedProduct == null)
            {
                MessageBox.Show(
                    "Vui lòng chọn sản phẩm cần xóa!",
                    "Thông báo",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Warning);

                return;
            }

            // TC05:
            // Yes / No + Question
            DialogResult result =
                MessageBox.Show(
                    "Bạn có chắc chắn muốn xóa sản phẩm:\n" +
                    selectedProduct.ProductName +
                    " ?",
                    "Xác nhận xóa",
                    MessageBoxButtons.YesNo,
                    MessageBoxIcon.Question);

            if (result == DialogResult.Yes)
            {
                products.Remove(
                    selectedProduct);

                selectedProduct = null;

                productBindingSource.ResetBindings(false);

                ClearInput();

                UpdateStatus();
            }
        }

        // =========================================================
        // LIVE SEARCH
        // =========================================================

        private void txtSearch_TextChanged(
            object sender,
            EventArgs e)
        {
            string keyword =
                txtSearch.Text
                    .Trim()
                    .ToLower();

            // Không nhập gì -> hiện toàn bộ
            if (string.IsNullOrWhiteSpace(
                keyword))
            {
                productBindingSource.DataSource =
                    products;

                dgvProducts.DataSource =
                    productBindingSource;

                return;
            }

            // Tìm theo tên
            List<Product> result =
                products
                    .Where(p =>
                        p.ProductName
                            .ToLower()
                            .Contains(keyword))
                    .ToList();

            productBindingSource.DataSource =
                new BindingList<Product>(
                    result);

            dgvProducts.DataSource =
                productBindingSource;

            dgvProducts.ClearSelection();
        }

        // =========================================================
        // EXPORT CSV
        // CTRL + E
        // =========================================================

        private void mnuExportCSV_Click(
            object sender,
            EventArgs e)
        {
            if (products.Count == 0)
            {
                MessageBox.Show(
                    "Danh sách sản phẩm đang trống!",
                    "Thông báo",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information);

                return;
            }

            using (SaveFileDialog saveFileDialog =
                   new SaveFileDialog())
            {
                saveFileDialog.Title =
                    "Xuất danh sách sản phẩm";

                saveFileDialog.Filter =
                    "CSV File (*.csv)|*.csv";

                saveFileDialog.DefaultExt =
                    "csv";

                saveFileDialog.AddExtension =
                    true;

                saveFileDialog.FileName =
                    "TechMart_Products.csv";

                if (saveFileDialog.ShowDialog()
                    != DialogResult.OK)
                {
                    return;
                }

                try
                {
                    using (StreamWriter writer =
                           new StreamWriter(
                               saveFileDialog.FileName,
                               false,
                               new UTF8Encoding(true)))
                    {
                        // Header CSV
                        writer.WriteLine(
                            "Mã SP,Tên SP,Danh Mục,Đơn Giá,Số Lượng");

                        foreach (Product p
                                 in products)
                        {
                            writer.WriteLine(
                                ToCsv(p.ProductId)
                                + ","
                                + ToCsv(p.ProductName)
                                + ","
                                + ToCsv(p.CategoryName)
                                + ","
                                + p.UnitPrice
                                + ","
                                + p.Quantity);
                        }
                    }

                    MessageBox.Show(
                        "Xuất CSV thành công!",
                        "Thông báo",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Information);
                }
                catch (Exception ex)
                {
                    MessageBox.Show(
                        "Không thể xuất file!\n" +
                        ex.Message,
                        "Lỗi",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Error);
                }
            }
        }

        // =========================================================
        // XỬ LÝ CSV
        // =========================================================

        private string ToCsv(string value)
        {
            if (value == null)
            {
                value = "";
            }

            value =
                value.Replace(
                    "\"",
                    "\"\"");

            return "\"" + value + "\"";
        }

        // =========================================================
        // EXIT
        // CTRL + X
        // =========================================================

        private void mnuExit_Click(
            object sender,
            EventArgs e)
        {
            DialogResult result =
                MessageBox.Show(
                    "Bạn có chắc chắn muốn thoát chương trình?",
                    "Xác nhận",
                    MessageBoxButtons.YesNo,
                    MessageBoxIcon.Question);

            if (result ==
                DialogResult.Yes)
            {
                Application.Exit();
            }
        }

        // =========================================================
        // XÓA DỮ LIỆU Ô NHẬP
        // =========================================================

        private void ClearInput()
        {
            txtProductId.Clear();

            txtProductName.Clear();

            txtUnitPrice.Clear();

            txtQuantity.Clear();

            cboCategory.SelectedIndex = -1;

            selectedImagePath = "";

            selectedProduct = null;

            if (picAvatar.Image != null)
            {
                picAvatar.Image.Dispose();

                picAvatar.Image = null;
            }

            errorProvider.Clear();

            dgvProducts.ClearSelection();

            txtProductId.Focus();
        }

        // =========================================================
        // STATUS STRIP
        // =========================================================

        private void UpdateStatus()
        {
            lblStatus.Text =
                "Tổng số sản phẩm: "
                + products.Count;
        }
    }
}
using System;
using System.ComponentModel;
using System.Drawing;
using System.Windows.Forms;
using static System.Net.Mime.MediaTypeNames;

namespace bai3
{
    partial class Form1
    {
        private IContainer components = null;

        private MenuStrip menuStrip1;
        private ToolStripMenuItem menuFile;
        private ToolStripMenuItem mnuExportCSV;
        private ToolStripMenuItem mnuExit;

        private StatusStrip statusStrip1;
        private ToolStripStatusLabel lblStatus;

        private TableLayoutPanel tableMain;

        private Panel pnlInput;
        private Panel pnlData;

        private Label lblTitleInput;
        private Label lblProductId;
        private Label lblProductName;
        private Label lblCategory;
        private Label lblUnitPrice;
        private Label lblQuantity;

        private TextBox txtProductId;
        private TextBox txtProductName;
        private TextBox txtUnitPrice;
        private TextBox txtQuantity;

        private ComboBox cboCategory;

        private PictureBox picAvatar;

        private Button btnChooseImage;
        private Button btnAdd;
        private Button btnUpdate;
        private Button btnDelete;

        private Label lblTitleData;
        private Label lblSearch;
        private TextBox txtSearch;

        private DataGridView dgvProducts;

        private DataGridViewTextBoxColumn colProductId;
        private DataGridViewTextBoxColumn colProductName;
        private DataGridViewTextBoxColumn colCategory;
        private DataGridViewTextBoxColumn colUnitPrice;
        private DataGridViewTextBoxColumn colQuantity;

        private ErrorProvider errorProvider;
        private BindingSource productBindingSource;

        protected override void Dispose(bool disposing)
        {
            if (disposing && components != null)
            {
                components.Dispose();
            }

            base.Dispose(disposing);
        }

        private void InitializeComponent()
        {
            components = new Container();

            menuStrip1 = new MenuStrip();
            menuFile = new ToolStripMenuItem();
            mnuExportCSV = new ToolStripMenuItem();
            mnuExit = new ToolStripMenuItem();

            statusStrip1 = new StatusStrip();
            lblStatus = new ToolStripStatusLabel();

            tableMain = new TableLayoutPanel();

            pnlInput = new Panel();
            pnlData = new Panel();

            lblTitleInput = new Label();
            lblProductId = new Label();
            lblProductName = new Label();
            lblCategory = new Label();
            lblUnitPrice = new Label();
            lblQuantity = new Label();

            txtProductId = new TextBox();
            txtProductName = new TextBox();
            txtUnitPrice = new TextBox();
            txtQuantity = new TextBox();

            cboCategory = new ComboBox();

            picAvatar = new PictureBox();

            btnChooseImage = new Button();
            btnAdd = new Button();
            btnUpdate = new Button();
            btnDelete = new Button();

            lblTitleData = new Label();
            lblSearch = new Label();
            txtSearch = new TextBox();

            dgvProducts = new DataGridView();

            colProductId = new DataGridViewTextBoxColumn();
            colProductName = new DataGridViewTextBoxColumn();
            colCategory = new DataGridViewTextBoxColumn();
            colUnitPrice = new DataGridViewTextBoxColumn();
            colQuantity = new DataGridViewTextBoxColumn();

            errorProvider = new ErrorProvider(components);
            productBindingSource = new BindingSource(components);

            // ==============================
            // MENU
            // ==============================

            menuStrip1.Items.AddRange(new ToolStripItem[]
            {
                menuFile
            });

            menuFile.Text = "File";

            menuFile.DropDownItems.AddRange(new ToolStripItem[]
            {
                mnuExportCSV,
                mnuExit
            });

            mnuExportCSV.Text = "Export CSV";
            mnuExportCSV.ShortcutKeys = Keys.Control | Keys.E;

            mnuExit.Text = "Exit";
            mnuExit.ShortcutKeys = Keys.Control | Keys.X;

            menuStrip1.Dock = DockStyle.Top;

            // ==============================
            // STATUS
            // ==============================

            lblStatus.Text = "Tổng số sản phẩm: 0";

            statusStrip1.Items.Add(lblStatus);
            statusStrip1.Dock = DockStyle.Bottom;

            // ==============================
            // TABLE MAIN
            // ==============================

            tableMain.Dock = DockStyle.Fill;

            tableMain.ColumnCount = 2;

            tableMain.ColumnStyles.Add(
                new ColumnStyle(SizeType.Percent, 35F));

            tableMain.ColumnStyles.Add(
                new ColumnStyle(SizeType.Percent, 65F));

            tableMain.RowCount = 1;

            tableMain.RowStyles.Add(
                new RowStyle(SizeType.Percent, 100F));

            tableMain.Padding = new Padding(10);

            tableMain.Controls.Add(pnlInput, 0, 0);
            tableMain.Controls.Add(pnlData, 1, 0);

            // ==============================
            // PANEL TRÁI
            // ==============================

            pnlInput.Dock = DockStyle.Fill;
            pnlInput.Padding = new Padding(15);
            pnlInput.BorderStyle = BorderStyle.FixedSingle;

            // Tiêu đề
            lblTitleInput.Text = "THÔNG TIN SẢN PHẨM";
            lblTitleInput.Font =
                new Font("Segoe UI", 14F, FontStyle.Bold);

            lblTitleInput.AutoSize = true;
            lblTitleInput.Location = new Point(20, 20);

            // Mã sản phẩm
            lblProductId.Text = "Mã sản phẩm";
            lblProductId.AutoSize = true;
            lblProductId.Location = new Point(20, 70);

            txtProductId.Location = new Point(20, 92);
            txtProductId.Size = new Size(300, 27);
            txtProductId.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Left |
                AnchorStyles.Right;

            // Tên sản phẩm
            lblProductName.Text = "Tên sản phẩm";
            lblProductName.AutoSize = true;
            lblProductName.Location = new Point(20, 130);

            txtProductName.Location = new Point(20, 152);
            txtProductName.Size = new Size(300, 27);
            txtProductName.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Left |
                AnchorStyles.Right;

            // Danh mục
            lblCategory.Text = "Danh mục";
            lblCategory.AutoSize = true;
            lblCategory.Location = new Point(20, 190);

            cboCategory.Location = new Point(20, 212);
            cboCategory.Size = new Size(300, 28);

            cboCategory.DropDownStyle =
                ComboBoxStyle.DropDownList;

            cboCategory.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Left |
                AnchorStyles.Right;

            // Đơn giá
            lblUnitPrice.Text = "Đơn giá";
            lblUnitPrice.AutoSize = true;
            lblUnitPrice.Location = new Point(20, 250);

            txtUnitPrice.Location = new Point(20, 272);
            txtUnitPrice.Size = new Size(300, 27);

            txtUnitPrice.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Left |
                AnchorStyles.Right;

            // Số lượng
            lblQuantity.Text = "Số lượng";
            lblQuantity.AutoSize = true;
            lblQuantity.Location = new Point(20, 310);

            txtQuantity.Location = new Point(20, 332);
            txtQuantity.Size = new Size(300, 27);

            txtQuantity.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Left |
                AnchorStyles.Right;

            // PictureBox
            picAvatar.Location = new Point(20, 375);
            picAvatar.Size = new Size(200, 125);

            picAvatar.BorderStyle =
                BorderStyle.FixedSingle;

            picAvatar.SizeMode =
                PictureBoxSizeMode.Zoom;

            // Chọn ảnh
            btnChooseImage.Text = "Chọn ảnh";
            btnChooseImage.Location =
                new Point(230, 420);

            btnChooseImage.Size =
                new Size(90, 35);

            // Các nút chức năng
            btnAdd.Text = "Thêm mới";
            btnAdd.Location = new Point(20, 520);
            btnAdd.Size = new Size(95, 40);

            btnUpdate.Text = "Cập nhật";
            btnUpdate.Location = new Point(125, 520);
            btnUpdate.Size = new Size(95, 40);

            btnDelete.Text = "Xóa";
            btnDelete.Location = new Point(230, 520);
            btnDelete.Size = new Size(90, 40);

            pnlInput.Controls.Add(lblTitleInput);

            pnlInput.Controls.Add(lblProductId);
            pnlInput.Controls.Add(txtProductId);

            pnlInput.Controls.Add(lblProductName);
            pnlInput.Controls.Add(txtProductName);

            pnlInput.Controls.Add(lblCategory);
            pnlInput.Controls.Add(cboCategory);

            pnlInput.Controls.Add(lblUnitPrice);
            pnlInput.Controls.Add(txtUnitPrice);

            pnlInput.Controls.Add(lblQuantity);
            pnlInput.Controls.Add(txtQuantity);

            pnlInput.Controls.Add(picAvatar);
            pnlInput.Controls.Add(btnChooseImage);

            pnlInput.Controls.Add(btnAdd);
            pnlInput.Controls.Add(btnUpdate);
            pnlInput.Controls.Add(btnDelete);

            // ==============================
            // PANEL PHẢI
            // ==============================

            pnlData.Dock = DockStyle.Fill;
            pnlData.Padding = new Padding(15);
            pnlData.BorderStyle = BorderStyle.FixedSingle;

            lblTitleData.Text =
                "DANH SÁCH SẢN PHẨM";

            lblTitleData.Font =
                new Font(
                    "Segoe UI",
                    14F,
                    FontStyle.Bold);

            lblTitleData.AutoSize = true;
            lblTitleData.Location =
                new Point(20, 20);

            lblSearch.Text = "Tìm kiếm:";
            lblSearch.AutoSize = true;
            lblSearch.Location =
                new Point(20, 72);

            txtSearch.Location =
                new Point(95, 68);

            txtSearch.Size =
                new Size(500, 27);

            txtSearch.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Left |
                AnchorStyles.Right;

            // ==============================
            // DATAGRIDVIEW
            // ==============================

            dgvProducts.Location =
                new Point(20, 115);

            dgvProducts.Size =
                new Size(650, 440);

            dgvProducts.Anchor =
                AnchorStyles.Top |
                AnchorStyles.Bottom |
                AnchorStyles.Left |
                AnchorStyles.Right;

            dgvProducts.AutoGenerateColumns = false;

            dgvProducts.SelectionMode =
                DataGridViewSelectionMode.FullRowSelect;

            dgvProducts.MultiSelect = false;

            dgvProducts.AllowUserToAddRows = false;

            dgvProducts.ReadOnly = true;

            dgvProducts.AutoSizeColumnsMode =
                DataGridViewAutoSizeColumnsMode.Fill;

            // Cột mã SP
            colProductId.HeaderText = "Mã SP";
            colProductId.DataPropertyName =
                "ProductId";

            // Cột tên SP
            colProductName.HeaderText =
                "Tên SP";

            colProductName.DataPropertyName =
                "ProductName";

            // Danh mục
            colCategory.HeaderText =
                "Danh Mục";

            colCategory.DataPropertyName =
                "CategoryName";

            // Đơn giá
            colUnitPrice.HeaderText =
                "Đơn Giá";

            colUnitPrice.DataPropertyName =
                "UnitPrice";

            colUnitPrice.DefaultCellStyle.Format =
                "N0";

            colUnitPrice.DefaultCellStyle.Alignment =
                DataGridViewContentAlignment.MiddleRight;

            // Số lượng
            colQuantity.HeaderText =
                "Số Lượng";

            colQuantity.DataPropertyName =
                "Quantity";

            dgvProducts.Columns.AddRange(
                colProductId,
                colProductName,
                colCategory,
                colUnitPrice,
                colQuantity);

            pnlData.Controls.Add(lblTitleData);
            pnlData.Controls.Add(lblSearch);
            pnlData.Controls.Add(txtSearch);
            pnlData.Controls.Add(dgvProducts);

            // ==============================
            // ERROR PROVIDER
            // ==============================

            errorProvider.ContainerControl = this;

            // ==============================
            // FORM
            // ==============================

            AutoScaleDimensions =
                new SizeF(8F, 20F);

            AutoScaleMode =
                AutoScaleMode.Font;

            ClientSize =
                new Size(1200, 680);

            MinimumSize =
                new Size(1000, 620);

            StartPosition =
                FormStartPosition.CenterScreen;

            Text =
                "TechMart Product Manager";

            Controls.Add(tableMain);
            Controls.Add(statusStrip1);
            Controls.Add(menuStrip1);

            MainMenuStrip = menuStrip1;
        }
    }
}
namespace bai3
{
    public class Product
    {
        public string ProductId { get; set; } = "";
        public string ProductName { get; set; } = "";

        public int CategoryId { get; set; }
        public string CategoryName { get; set; } = "";

        public decimal UnitPrice { get; set; }
        public int Quantity { get; set; }

        public string ImagePath { get; set; } = "";
    }
}
namespace bai3
{
    public class Category
    {
        public int Id { get; set; }

        public string Name { get; set; } = "";
    }
}
