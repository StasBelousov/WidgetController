//
//  MainWidgetSettingModuleViewController.swift
//  B2C 3.0
//
//  Created by Stanislav Belousov on 02/07/24.
//

import UIKit
import B2C3UIKit
import B2C3Core
import B2CHelpers

final class MainWidgetSettingModuleViewController: BaseViewController {

    var presenter: MainWidgetSettingModulePresenterProtocol?

    // MARK: - UI Elements

    private lazy var descriptionLabel: UILabel = {
        let label = UILabel()
        label.font = .bodyMedium
        label.textColor = Palette.main(.text)
        label.text = Constants.settingDescription.localized()
        label.numberOfLines = 0
        return label
    }()

    private let tableView: UITableView = {
        let tableView = UITableView()
        tableView.backgroundColor = Palette.background(.primary)
        tableView.separatorStyle = .none
        return tableView
    }()

    // MARK: - Properties

    private var dataSource: [[String]] = {
        let defaultValue: [MainWidgetType] = [.transfers, .news, .exchangeRates, .history]
        let activeWidgets = AuthManager.shared.mainPageWidgetList.map { $0.title }
        let inactiveWidgets = defaultValue.map { $0.title }.filter { !activeWidgets.contains($0) }
        return [activeWidgets, inactiveWidgets + [Constants.addedEverythingKey.localized()]]
    }()

    private let feedbackGenerator = UISelectionFeedbackGenerator()
    private var lastDestinationIndexPath: IndexPath?
    private var hasChanges = false

    // MARK: - Lifecycle Methods

    override func viewDidLoad() {
        super.viewDidLoad()
        setupSubviews()
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        if hasChanges, let activeWidgets = dataSource.first, !activeWidgets.isEmpty {
            presenter?.saveMove(result: activeWidgets)
        }
    }

    // MARK: - Setup Methods

    override func setupSubviews() {
        super.setupSubviews()
        title = Constants.settingTitle.localized()
        view.backgroundColor = Palette.background(.primary)

        tableView.dataSource = self
        tableView.delegate = self
        tableView.dragDelegate = self
        tableView.dropDelegate = self
        tableView.dragInteractionEnabled = true
        tableView.isScrollEnabled = false
        tableView.register(WidgetSettingCell.self, forCellReuseIdentifier: WidgetSettingCell.identifier)

        view.addSubview(descriptionLabel)
        descriptionLabel.snp.makeConstraints { make in
            make.top.equalTo(view.safeAreaLayoutGuide).offset(Configs.BaseDimensions.Inset.inset16.rawValue)
            make.leading.trailing.equalToSuperview().inset(Configs.BaseDimensions.Inset.inset16.rawValue)
            make.height.equalTo(Constants.descriptionLabelHeight)
        }

        view.addSubview(tableView)
        tableView.snp.makeConstraints { make in
            make.top.equalTo(descriptionLabel.snp.bottom).offset(Configs.BaseDimensions.Inset.inset2.rawValue)
            make.leading.trailing.equalToSuperview()
            make.bottom.equalTo(view.safeAreaLayoutGuide)
        }
    }

    // MARK: - Helper Methods

    private func adjustedDestinationIndexPath(_ destinationIndexPath: IndexPath?) -> IndexPath? {
        var indexPath = destinationIndexPath

        if indexPath == nil {
            let row = dataSource[1].count
            indexPath = IndexPath(row: row, section: 1)
        }

        if indexPath!.section == 1 && indexPath!.row == 0 {
            indexPath = IndexPath(row: 1, section: 1)
        }

        return indexPath
    }

    private func isForbiddenItem(_ itemTitle: String, destinationSection: Int) -> Bool {
        return itemTitle == Constants.newsAndPromotionsKey.localized() && destinationSection == 1
    }

    private func showForbiddenMoveAlertIfNeeded() {
        if view.viewWithTag(Constants.animationViewTag) == nil {
            showAnimatedAlertView(with: ImagesIconsHelper.notifications(.attention24), text: Constants.blockCannotHiddenKey.localized(), in: self.view)
        }
    }

    // MARK: - Animated Alert View

    func showAnimatedAlertView(with image: UIImage, text: String, in parentView: UIView) {
        let containerView = UIView()
        containerView.alpha = 0.0
        containerView.transform = CGAffineTransform(scaleX: 0.5, y: 0.5)
        containerView.backgroundColor = Palette.system(.attentionOpacity)
        containerView.tag = Constants.animationViewTag
        containerView.roundCorners(.allCorners, radius: Configs.BaseDimensions.CornerRadius.radius12.rawValue)

        let imageView = UIImageView(image: image)
        imageView.contentMode = .scaleAspectFit
        containerView.addSubview(imageView)

        let label = UILabel()
        label.text = text
        label.textAlignment = .left
        label.font = .bodyMedium
        label.textColor = Palette.main(.text)
        label.numberOfLines = 1
        containerView.addSubview(label)

        parentView.addSubview(containerView)

        containerView.snp.makeConstraints { make in
            make.bottom.equalTo(view.safeAreaLayoutGuide)
            make.leading.trailing.equalToSuperview().inset(Configs.BaseDimensions.Inset.inset16.rawValue)
            make.height.equalTo(Constants.animatedViewHeight)
        }

        imageView.snp.makeConstraints { make in
            make.leading.equalToSuperview().offset(Configs.BaseDimensions.Inset.inset8.rawValue)
            make.centerY.equalToSuperview()
            make.width.height.equalTo(24)
        }

        label.snp.makeConstraints { make in
            make.leading.equalTo(imageView.snp.trailing).offset(8)
            make.trailing.equalToSuperview().offset(-Configs.BaseDimensions.Inset.inset8.rawValue)
            make.top.bottom.equalToSuperview()
        }

        UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.6, initialSpringVelocity: 0.5, options: .curveEaseInOut, animations: { [weak self] in
            containerView.alpha = 1.0
            containerView.transform = .identity
        }, completion: nil)
    }

    func hideAnimatedView(_ viewToHide: UIView, completion: (() -> Void)? = nil) {
        UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.5, options: .curveEaseInOut, animations: {
            viewToHide.alpha = 0.0
            viewToHide.transform = CGAffineTransform(scaleX: 0.1, y: 0.1)
        }, completion: { _ in
            viewToHide.removeFromSuperview()
            completion?()
        })
    }

    // MARK: - Constants

    private enum Constants {
        static let newsAndPromotionsKey = "Main_NewsAndPromotions"
        static let addedEverythingKey = "Main_Widgets_AddedEveryThing"
        static let blockCannotHiddenKey = "Main_Widgets_BlockCannotHidden"
        static let settingDescription = "Main_SettingDescription"
        static let settingTitle = "Main_Setting"
        static let animationViewTag = 1
        static let rowHeight: CGFloat = 70.0
        static let descriptionLabelHeight: CGFloat = 60.0
        static let animatedViewHeight: CGFloat = 44.0
    }
}

// MARK: - MainWidgetSettingModuleViewProtocol

extension MainWidgetSettingModuleViewController: MainWidgetSettingModuleViewProtocol {}

// MARK: - UITableViewDataSource

extension MainWidgetSettingModuleViewController: UITableViewDataSource {

    func numberOfSections(in tableView: UITableView) -> Int {
        return dataSource.count
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return dataSource[section].count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

        guard let cell = tableView.dequeueReusableCell(withIdentifier: WidgetSettingCell.identifier, for: indexPath) as? WidgetSettingCell else {
            return UITableViewCell()
        }
        cell.configure(with: dataSource[indexPath.section][indexPath.row])
        return cell
    }
}

// MARK: - UITableViewDelegate

extension MainWidgetSettingModuleViewController: UITableViewDelegate {

    func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        let sectionNames = [
            NSLocalizedString("Label_Active".localized(), comment: ""),
            NSLocalizedString("Label_Available".localized(), comment: "")
        ]
        return section < sectionNames.count ? sectionNames[section] : nil
    }

    func tableView(_ tableView: UITableView, willDisplayHeaderView view: UIView, forSection section: Int) {
        if let headerView = view as? UITableViewHeaderFooterView {
            headerView.contentView.backgroundColor = Palette.background(.primary)
            headerView.textLabel?.textColor = Palette.main(.text)
            headerView.textLabel?.font = .titleMedium
            headerView.textLabel?.numberOfLines = 0

            headerView.textLabel?.snp.makeConstraints { make in
                make.leading.trailing.equalToSuperview().inset(Configs.BaseDimensions.Inset.inset16.rawValue)
            }
        }
    }

    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        let targetText = Constants.addedEverythingKey.localized()

        if dataSource[1].count > 1 {
            let index = IndexPath(row: dataSource[1].firstIndex(of: targetText) ?? 0, section: 1)
            if indexPath == index {
                return 0
            }
        }
        return Constants.rowHeight
    }

    func tableView(_ tableView: UITableView, viewForFooterInSection section: Int) -> UIView? {
        if section == 1 {
            let footerView = UIView()
            footerView.backgroundColor = .clear
            return footerView
        }
        return nil
    }

    func tableView(_ tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
        if section == 1 {
            let tableViewHeight = tableView.frame.height
            let contentHeight = tableView.contentSize.height
            let footerHeight = max(0, tableViewHeight - contentHeight)
            return footerHeight
        }
        return 0
    }
}

// MARK: - UITableViewDragDelegate

extension MainWidgetSettingModuleViewController: UITableViewDragDelegate {

    func tableView(_ tableView: UITableView, itemsForBeginning session: UIDragSession, at indexPath: IndexPath) -> [UIDragItem] {
        let nonMovableItem = Constants.addedEverythingKey.localized()
        let item = dataSource[indexPath.section][indexPath.row]
        if item == nonMovableItem {
            return []
        }

        let itemProvider = NSItemProvider(object: item as NSString)
        let dragItem = UIDragItem(itemProvider: itemProvider)
        dragItem.localObject = item

        feedbackGenerator.selectionChanged()
        feedbackGenerator.prepare()

        return [dragItem]
    }
}

// MARK: - UITableViewDropDelegate

extension MainWidgetSettingModuleViewController: UITableViewDropDelegate {

    func tableView(_ tableView: UITableView, performDropWith coordinator: UITableViewDropCoordinator) {
        guard let destinationIndexPath = adjustedDestinationIndexPath(coordinator.destinationIndexPath) else { return }

        tableView.performBatchUpdates({
            for item in coordinator.items {
                if let sourceIndexPath = item.sourceIndexPath,
                   let itemTitle = item.dragItem.localObject as? String {

                    if isForbiddenItem(itemTitle, destinationSection: destinationIndexPath.section) {
                        showForbiddenMoveAlertIfNeeded()
                        return
                    }

                    let movedItem = dataSource[sourceIndexPath.section].remove(at: sourceIndexPath.row)
                    dataSource[destinationIndexPath.section].insert(movedItem, at: destinationIndexPath.row)
                    hasChanges = true

                    tableView.moveRow(at: sourceIndexPath, to: destinationIndexPath)
                }
            }
        }, completion: nil)

        coordinator.drop(coordinator.items.first!.dragItem, toRowAt: destinationIndexPath)
        feedbackGenerator.selectionChanged()
    }

    func tableView(_ tableView: UITableView, canHandle session: UIDropSession) -> Bool {
        return session.canLoadObjects(ofClass: NSString.self)
    }

    func tableView(_ tableView: UITableView, dropSessionDidUpdate session: UIDropSession, withDestinationIndexPath destIndexPath: IndexPath?) -> UITableViewDropProposal {
        if session.localDragSession != nil {
            guard let destinationIndexPath = adjustedDestinationIndexPath(destIndexPath) else {
                return UITableViewDropProposal(operation: .cancel)
            }

            let draggedItems = session.items.compactMap { $0.localObject as? String }
            let isForbiddenItemMoving = draggedItems.contains { isForbiddenItem($0, destinationSection: destinationIndexPath.section) }

            if isForbiddenItemMoving {
                showForbiddenMoveAlertIfNeeded()
            }

            if lastDestinationIndexPath != destinationIndexPath {
                lastDestinationIndexPath = destinationIndexPath
                feedbackGenerator.selectionChanged()
                feedbackGenerator.prepare()
            }

            return UITableViewDropProposal(operation: .move, intent: .insertAtDestinationIndexPath)
        } else {
            return UITableViewDropProposal(operation: .forbidden)
        }
    }

    func tableView(_ tableView: UITableView, dropSessionDidEnd session: UIDropSession) {
        feedbackGenerator.prepare()
        lastDestinationIndexPath = nil
            if let foundView = self.view.viewWithTag(Constants.animationViewTag) {
                self.hideAnimatedView(foundView)
        }
    }
}
